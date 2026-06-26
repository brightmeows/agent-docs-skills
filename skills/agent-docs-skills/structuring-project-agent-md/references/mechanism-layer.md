# 机制层——Hooks / Subagents / Rules / Plugins

> `structuring-project-agent-md` 参考文件。
> AGENTS.md / CLAUDE.md / `.cursor/rules` 都是**指令层**——告诉代理该做什么，依赖模型遵从。
> Claude Code 还提供**机制层**——确定性强制代理行为或隔离执行。
> 这是通用原则 "An instruction asks, a mechanism requires"（见 `writing-agent-docs` A.2）在 Claude Code 的落地。

---

## 指令层 vs 机制层 vs 隔离层

| 层 | 机制 | 强制度 | 上下文成本 |
|---|---|---|---|
| **指令层** | AGENTS.md / CLAUDE.md / rules | 依赖模型遵从（压力/长会话/注入可绕过）| 高（常驻）|
| **机制层** | Hooks / Permissions | 确定性（exit 2 阻断，不可绕过）| 低（配置在上下文外）|
| **隔离层** | Subagents | 独立上下文执行 | 低（仅摘要回主会话）|

**关键判据**：若一条规则"绝对不能被违反"（提交密钥、force push、删生产数据），指令是错的工具——模型在长会话、时间压力、或被注入的文件内容诱导下会失败。真正的护栏必须确定性，即 **hooks 与 permissions**。

---

## 七种指令方法决策表

来源：[Steering Claude Code](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more)（Anthropic 官方博客，2026-06）。

| 方法 | 加载时机 | 压缩行为 | 上下文成本 | 适用场景 |
|---|---|---|---|---|
| CLAUDE.md（根）| 会话开始常驻 | 压缩后重读 | 高（每行都耗 token）| 构建命令、目录布局、monorepo 结构、团队约定 |
| CLAUDE.md（子目录）| 读到该目录文件时按需 | 触碰后丢失直到再次触碰 | 低 | 子目录专属约定 |
| `.claude/rules`（无 `paths`）| 会话开始常驻 | 压缩后重注入 | 中（等同 CLAUDE.md）| 避免——浪费 token |
| `.claude/rules`（带 `paths:`）| 路径匹配时 | 压缩后重注入 | 中 | 文件级约束（如所有 API handler 必须用 Zod 校验）|
| Skills | name+description 常驻；body 调用时加载 | 重注入到共享预算，最旧先弃 | 低 | 程序化工作流（部署/发布清单/审查流程）|
| Subagents | name+description+工具表常驻；body 仅调用时加载 | 仅最终消息回主会话 | 低（隔离上下文）| 隔离侧任务（深度搜索、日志分析、依赖审计）|
| **Hooks** | 生命周期事件触发 | **完全绕过压缩** | 低（配置在上下文外）| **确定性自动化**：跑 linter、阻断命令、压缩前备份、Slack 通知 |
| Output styles | 会话开始注入系统提示 | 永不压缩 | 高（**覆盖**默认系统提示）| 角色大改（慎用，见下）|
| append-system-prompt | 调用时 CLI flag 传入 | 仅当次调用 | 中（缓存后降低）| 语气/格式/领域知识，追加而非替换 |

### 反模式映射（信号 → 正确位置）

官方博客明确给出"出现以下信号时该换位置"：

| 信号 | 错误位置 | 正确位置 |
|---|---|---|
| "每次 X，总要做 Y" | CLAUDE.md | **Hook**（`PostToolUse` 跑 formatter / `Stop` 发通知）|
| "绝不做 X" | CLAUDE.md | **Hook + permissions**（`PreToolUse` exit 2 阻断）|
| 30 行流程写进配置 | CLAUDE.md | **Skill**（body 按需加载）|
| API 专属规则无 paths | 无 glob 的 rule | 带 `paths:` 的 **rule**（或 `.cursor/rules` globs）|
| 个人偏好 | 项目级 CLAUDE.md | 个人级文件（见 `structuring-personal-agent-md`）|

---

## Hooks（确定性强制）

Hooks 是用户定义的命令 / HTTP / LLM 判断，在 Claude Code 生命周期事件上**确定性**触发。完整规范见 [hooks-guide](https://code.claude.com/docs/en/hooks-guide) 与 [hooks reference](https://code.claude.com/docs/en/hooks)。

### 五种 hook 类型

| 类型 | 行为 | 适用 |
|---|---|---|
| `command` | 跑 shell 命令 | 确定性规则（format、block、log）|
| `http` | POST 事件数据到 URL | 共享审计服务、云端函数 |
| `mcp_tool` | 调已连接 MCP server 的工具 | 复用 MCP 能力 |
| `prompt` | 单轮 LLM 判断（默认 Haiku）| 需判断而非死规则（返回 `{ok, reason}`）|
| `agent` | 多轮验证（可读文件/跑命令，≤50 轮）| 需核对代码库实际状态 |

`command` / `http` / `mcp_tool` 确定性执行；`prompt` / `agent` 用模型判断。

### 关键能力——阻断工具调用

`PreToolUse` hook 读取 stdin JSON，**exit 2 即阻断**，stderr 反馈给代理让其调整：

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')
if echo "$COMMAND" | grep -q "drop table"; then
  echo "Blocked: 禁止删表" >&2   # stderr 成代理的反馈
  exit 2                          # exit 2 = 阻断
fi
exit 0   # exit 0 = 无意见，正常权限流程继续
```

或用结构化 JSON（exit 0 + stdout）做更细控制：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "改用 rg 而非 grep"
  }
}
```

`permissionDecision` 取值：`allow`（跳过交互提示，但 deny 规则仍生效）/ `deny`（取消调用并把 reason 喂回代理）/ `ask`（照常弹权限对话框）。

**`deny` 即使在 `bypassPermissions` 模式或 `--dangerously-skip-permissions` 下也生效**——这是组织级强制护栏的唯一可靠方式。反之 hook 的 `allow` 不能绕过 deny 规则（hook 只能收紧，不能放松）。

### 注册位置（决定作用域）

| 位置 | 作用域 | 可共享（进 git）|
|---|---|---|
| `~/.claude/settings.json` | 所有项目 | 否（本机）|
| `.claude/settings.json` | 单项目 | 是 |
| `.claude/settings.local.json` | 单项目 | 否（gitignore）|
| Managed policy settings | 组织级 | 是（admin 控制，不可被本地覆盖）|
| Plugin `hooks/hooks.json` | 插件启用时 | 是（随插件分发）|
| **Skill / agent frontmatter** | 技能/agent 激活期间 | 是（写在组件文件里）|

最后一行呼应 `writing-skill-md` 的 `hooks` frontmatter 字段——技能可自带生命周期钩子。

### 常用生命周期事件（节选）

完整表见 [hooks reference](https://code.claude.com/docs/en/hooks#hook-lifecycle)。最常用：

| 事件 | 触发 | 典型用途 |
|---|---|---|
| `PreToolUse` | 工具调用前 | 阻断危险命令、改写参数 |
| `PostToolUse` | 工具成功后 | 自动 format、记日志 |
| `SessionStart` | 会话开始/恢复/压缩后 | 注入动态上下文（如 `git log --oneline -5`）|
| `PreCompact` / `PostCompact` | 压缩前后 | 备份对话、重注入关键信息 |
| `Stop` | 代理结束响应时 | 验证任务真的完成（接 prompt/agent hook）|
| `Notification` | 等待输入时 | 桌面通知 |
| `InstructionsLoaded` | CLAUDE.md/rules 加载时 | 审计配置加载 |
| `ConfigChange` | 配置文件被外部改动 | 合规审计、阻断未授权改动 |

---

## Subagents（隔离执行）

详见 [subagents 文档](https://code.claude.com/docs/en/sub-agents)。Subagent 是 `.claude/agents/` 下的 markdown 文件（YAML frontmatter `name` / `description` / 可选 `model` / 工具限定 + body 作系统提示），在**独立上下文窗口**运行。
仅最终消息（摘要 + 元数据）回主会话。内置有 `Explore` / `Plan` / `general-purpose`。

### Subagent vs Skill 决策

| 特征 | Skill | Subagent |
|---|---|---|
| 执行位置 | 主线程（可见可控）| 隔离上下文 |
| 上下文污染 | 过程全进主会话 | 仅摘要回主会话 |
| 适用 | 要看着/干预每一步的流程 | 侧任务（深度搜索、日志分析、依赖审计）|
| 嵌套 | — | 可嵌套至 5 层；dynamic workflows 可编排数十到数百个 |

**判据**：中间结果你之后还要引用 → skill；中间结果是噪声、只要结论 → subagent。

---

## .claude/rules（路径限定规则）

`.claude/rules/*.md` 用 YAML frontmatter 的 `paths:` 字段限定加载范围——Claude Code 原生等价于 `.cursor/rules` 的 globs：

```yaml
---
paths:
  - "src/api/**"
  - "**/*.handler.ts"
---
所有 API handler 必须先用 Zod 校验输入。
```

无 `paths` 的 rule 等同 CLAUDE.md（常驻、耗 token），应避免。横切多个（但非全部）目录的约束（如"migrations 是 append-only"）适合 path-scoped rule；仅子目录专属的约定适合子目录 CLAUDE.md。

---

## Output styles 与 append-system-prompt（慎用）

- **Output styles**（`.claude/output-styles/`）注入系统提示，**永不压缩、加载权重最高**——但**会覆盖默认系统提示**（除非 frontmatter 设 `keep-coding-instructions: true`）。
  误用会让 Claude 从"软件工程师助手"退化为"通用助手"，丢失变更范围控制、注释时机、验证习惯等默认指令。优先用内置的 `Proactive` / `Explanatory` / `Learning`。
- **append-system-prompt**（`--append-system-prompt` CLI flag）是**追加**而非替换，仅当次调用生效，更安全。适合领域知识、格式偏好。但收益递减——指令越多遵从越松，矛盾指令尤甚。

---

## Plugins（打包分发）

[Plugins](https://code.claude.com/docs/en/plugins) 把 skills + agents + hooks + MCP servers + output styles 打包成可分发单元（`plugin.json` + 组件目录）。团队级一致环境（一套 hooks + 一套 skills + 共享 MCP）适合做成 plugin 跨项目复用。

---

## 与本仓库原则的映射

| 仓库原则 | 机制层落地 |
|---|---|
| `writing-agent-docs` A.2「确定性约束优先」| 能用 `PreToolUse` hook 阻断的，不写进 AGENTS.md |
| `writing-agent-docs` C.5「工作流闭包——完成标准」| `Stop` 的 prompt/agent hook 验证"真的完成"而非代理自报 |
| `structuring-project` 三层边界 `Never` | `Never` 类禁令 → hook + permissions（确定性兜底）|
| `structuring-project` Toolchain First | hook 是 Claude Code 原生工具链的一部分 |
| `writing-skill-md` 安全考虑 | `allowed-tools` 最小权限 + hook 收紧 = 纵深防御 |
