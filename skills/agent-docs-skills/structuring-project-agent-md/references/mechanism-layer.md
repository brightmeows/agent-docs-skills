# 机制层——Hooks / Subagents / Rules / Plugins / Dynamic Workflows

> `structuring-project-agent-md` 参考文件。
> **本文件描述的机制层以 Claude Code 为代表**——hooks、subagents、output styles、plugins、dynamic workflows 是 **Claude Code 专属**；
> 其它代理（OpenCode / Cursor / Gemini CLI / Copilot）有各自的等价或尚无等价机制，见 [跨工具支持矩阵](#跨工具支持矩阵)。
>
> AGENTS.md / CLAUDE.md / `.cursor/rules` 都是**指令层**——告诉代理该做什么，依赖模型遵从。
> 机制层则**确定性强制**代理行为或隔离执行。
> 这是通用原则 “An instruction asks, a mechanism requires”（见 `writing-agent-docs` A.2）的具体落地形态之一（Claude Code 形态）。

---

## 目录

- [指令层 vs 机制层 vs 隔离层](#指令层-vs-机制层-vs-隔离层)
- [跨工具支持矩阵](#跨工具支持矩阵)
- [八种指令方法决策表](#八种指令方法决策表)
- [Hooks（确定性强制 · Claude Code 专属）](#hooks确定性强制--claude-code-专属)
- [Subagents（隔离执行 · Claude Code 专属）](#subagents隔离执行--claude-code-专属)
- [Dynamic Workflows（动态执行 harness · Claude Code 专属）](#dynamic-workflows动态执行-harness--claude-code-专属)
- [路径限定规则（Claude Code `.claude/rules` · Cursor `.cursor/rules`）](#路径限定规则claude-code-clauderules--cursor-cursorrules)
- [Output styles 与 append-system-prompt（慎用 · Claude Code 专属）](#output-styles-与-append-system-prompt慎用--claude-code-专属)
- [Plugins（打包分发 · Claude Code 专属）](#plugins打包分发--claude-code-专属)
- [与本仓库原则的映射](#与本仓库原则的映射)
- [参考文献](#参考文献)

## 指令层 vs 机制层 vs 隔离层

| 层 | 机制 | 强制度 | 上下文成本 |
|---|---|---|---|
| **指令层** | AGENTS.md / CLAUDE.md / rules | 依赖模型遵从（压力/长会话/注入可绕过）| 高（常驻）|
| **机制层** | Hooks / Permissions | 确定性（exit 2 阻断，不可绕过）| 低（配置在上下文外）|
| **隔离层** | Subagents | 独立上下文执行 | 低（仅摘要回主会话）|
| **动态层** | Dynamic Workflows | 按需生成 harness，独立编排 | 取决于任务复杂度 |

**关键判据**：若一条规则“绝对不能被违反”（提交密钥、force push、删生产数据），指令是错的工具——模型在长会话、时间压力、或被注入的文件内容诱导下会失败。真正的护栏必须确定性，即 **hooks 与 permissions**。

---

## 跨工具支持矩阵

机制层以 Claude Code 为代表。下表标注各主流代理的支持情况（✅ 原生 / ◐ 有等价或部分 / ❌ 无），帮助判断某条指导是否跨工具可移植：

| 机制 | Claude Code | OpenCode | Cursor | Gemini CLI | Copilot |
|---|---|---|---|---|---|
| **Hooks**（生命周期 hook、exit 2 阻断）| ✅ 专属（5 类型、30+ 事件）| ❌（靠 permission / MCP / pre-commit）| ❌（靠 rules）| ❌ | ❌ |
| **Subagents**（隔离上下文、Agent 工具）| ✅ 专属（`.claude/agents/`、5 层嵌套）| ◐（agent 定义 + task 工具，概念相近）| ◐（composer / agent 模式）| ❌ | ❌ |
| **路径限定规则** | ✅ `.claude/rules` `paths:` | ◐（兼容 `.cursor/rules/*.mdc`）| ✅ `.cursor/rules` `globs` | ❌ | ❌ |
| **Output styles**（覆盖系统提示）| ✅ 专属 | ❌ | ❌ | ❌ | ❌ |
| **Plugins**（`plugin.json` 打包）| ✅ 专属 | ◐（自有 plugin / 扩展概念，格式不同）| ◐（extensions）| ❌ | ❌ |

**对照**——指令层与技能层是跨工具的：

| 层 | 跨工具标准 |
|---|---|
| AGENTS.md | ✅ 全部主流代理读取（跨工具 fallback）|
| SKILL.md | ✅ Agent Skills 开放标准（Claude Code / OpenCode / Cursor 原生；实现程度不一）|

> **写跨工具配置时**：机制层指导应明确标注工具归属；不确定某代理是否支持时，优先用跨工具标准（AGENTS.md 表达意图、SKILL.md 按需加载），把 Claude Code 专属机制作为“可选增强”而非依赖。完整概念对照见 [cross-tool-compat.md](./cross-tool-compat.md)。

### OpenCode 权限模式（参考）

OpenCode 不与 Claude Code hooks 直接对标，但提供了等效的 **`permission.skill`** 机制用于技能级访问控制：

- **三态控制**：`allow`（立即加载）/ `deny`（隐藏拒绝）/ `ask`（请求批准）
- **通配符支持**：如 `internal-*` 批量管理同模式技能
- **每 agent 覆盖**：可在 `opencode.json` 或 Markdown agent 定义中按 agent 覆写全局设置
- **实际用途**：阻止来源不明的技能自动加载、限制生产环境技能权限、按团队角色分配不同的技能可见性

示例：

```json
{
  "permission": {
    "skill": {
      "internal-deploy-*": "allow",
      "*": "ask"
    }
  }
}
```

参考：[opencode.ai/docs/skills](https://opencode.ai/docs/skills)

> **工具归属**：`permission.skill` 是 OpenCode 专属，不跨工具。写跨工具配置时不要依赖此机制。

---

## 八种指令方法决策表

来源：[Steering Claude Code](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more)（Anthropic 官方博客，2026-06）；[A Harness for Every Task](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code)（2026-06）。

| 方法 | 加载时机 | 压缩行为 | 上下文成本 | 适用场景 |
|---|---|---|---|---|
| CLAUDE.md（根）| 会话开始常驻 | 压缩后重读 | 高（每行都耗 token）| 构建命令、目录布局、monorepo 结构、团队约定 |
| CLAUDE.md（子目录）| 读到该目录文件时按需 | 触碰后丢失直到再次触碰 | 低 | 子目录专属约定 |
| `.claude/rules`（无 `paths`）| 会话开始常驻 | 压缩后重注入 | 中（等同 CLAUDE.md）| 避免——浪费 token |
| `.claude/rules`（带 `paths:`）| 路径匹配时 | 压缩后重注入 | 中 | 文件级约束（如所有 API handler 必须用 Zod 校验）|
| Skills | name+description 常驻；body 调用时加载 | 重注入到共享预算，最旧先弃 | 低 | 程序化工作流（部署/发布清单/审查流程）|
| Subagents | name+description+工具表常驻；body 仅调用时加载 | 仅最终消息回主会话 | 低（隔离上下文）| 隔离侧任务（深度搜索、日志分析、依赖审计）|
| **Hooks** | 生命周期事件触发 | **完全绕过压缩** | 低（配置在上下文外）| **确定性自动化**：跑 linter、阻断命令、压缩前备份、Slack 通知 |
| **Dynamic Workflows** | 调用时动态生成 | 按任务复杂度 | 取决于任务（复杂任务需更多 token）| 复杂、多步骤、需动态编排的任务（研究、安全分析、agent 团队、代码审查）|
| Output styles | 会话开始注入系统提示 | 永不压缩 | 高（**覆盖**默认系统提示）| 角色大改（慎用，见下）|
| append-system-prompt | 调用时 CLI flag 传入 | 仅当次调用 | 中（缓存后降低）| 语气/格式/领域知识，追加而非替换 |

### 反模式映射（信号 → 正确位置）

官方博客明确给出“出现以下信号时该换位置”：

| 信号 | 错误位置 | 正确位置 |
|---|---|---|
| “每次 X，总要做 Y” | CLAUDE.md | **Hook**（`PostToolUse` 跑 formatter / `Stop` 发通知）|
| “绝不做 X” | CLAUDE.md | **Hook + permissions**（`PreToolUse` exit 2 阻断）|
| 30 行流程写进配置 | CLAUDE.md | **Skill**（body 按需加载）|
| API 专属规则无 paths | 无 glob 的 rule | 带 `paths:` 的 **rule**（或 `.cursor/rules` globs）|
| 个人偏好 | 项目级 CLAUDE.md | 个人级文件（见 `structuring-personal-agent-md`）|
| 复杂编排任务需动态定制 | 静态 Skill / CLAUDE.md | **Dynamic Workflow**（按需生成 harness）|

---

## Hooks（确定性强制 · Claude Code 专属）

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

## Subagents（隔离执行 · Claude Code 专属）

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

## Dynamic Workflows（动态执行 harness · Claude Code 专属）

Dynamic Workflows 是 Claude Code 的 GA 功能（自 v2.1.154 起），Claude 可在运行时动态生成 JavaScript 编排脚本，将任务拆分为数十到数百个并行的 subagent，在后台执行的同时主会话保持响应（[R1]；[R2]）。

> **激活方式**：在提示中包含 `ultracode` 关键词，或设 `/effort ultracode` 让 Claude 自动为每个实质任务编排 workflow。保存后以 `/<name>` 命令复用。

### 与静态方法的区别

| 维度 | 静态（Skill / Subagent） | Dynamic Workflow |
|------|------------------------|------------------|
| 定义时机 | 编写时预定义 | 运行时动态生成 |
| 适用范围 | 通用、预知的工作流 | 复杂、多变、需定制编排的任务 |
| 灵活性 | 固定结构，需覆盖所有边缘情况 | 按需生成，task-specific |
| token 开销 | 可预测 | 取决于任务复杂度（复杂任务更多） |
| 适用场景 | 部署流程、代码审查、测试 | 代码库审计、大规模迁移、跨源研究、多角度计划 |
| 重复性 | 同一定义可反复使用 | 保存后也可复用（存为 `/<name>` 命令）|

### 常见模式

| 模式 | 说明 |
|------|------|
| **Fan-out / Synthesize** | 拆分为多子任务，各 agent 独立执行，汇总结果（屏障等待）|
| **对抗性交叉验证** | 多个独立 agent 相互审校对方发现，仅报告幸存结论 |
| **/loop + /goal** | 重复执行工作流+硬性完成条件（适合 triage、研究、验证）|
| **Token 预算** | 为 workflow 设置显式 token 预算（如 “use 10k tokens”）|

### 与 Skill 的协同（[R1]）

Skill 和 Dynamic Workflow 互补而非替代：

| 协同模式 | 做法 | 示例 |
|---------|------|------|
| **Skill 分发 Workflow 模板** | 将 `.js` 工作流文件放入 skill 目录，在 SKILL.md 中引用为 template | 部署流程 skill 附带回滚 workflow |
| **Skill 定义 Workflow 子任务** | Workflow 中引用的子任务由 Skill 提供精确指令 | 审计 workflow 引用 `code-review` skill |
| **Skill 作为 Fallback** | DW 不适合的简单任务回退到 Skill 按需加载 | 复杂迁移用 DW，单文件修改用 Skill |

**关键区分**：Skill 是代理遵从的**指令集**，DW 是可编排 agent 的**脚本**。Skill 的内容加载到代理的上下文，DW 的控制流在运行时脚本中。两者协作时：DW 决定"谁做什么"，Skill 告诉每个 agent "怎么做"。

### 适用判据

| 适合 Dynamic Workflow | 适合 Skill / Subagent |
|-----------------------|----------------------|
| 任务结构多变，无法预定义 | 工作流稳定、可预定义 |
| 需要动态决定执行路径 | 执行路径确定 |
| 希望 Claude 自行设计 harness | 希望人为控制每一步 |
| 任务边界不清晰 | 任务边界明确 |
| 需编排 10+ agent | 1–5 个 agent 足够 |

---

## 路径限定规则（Claude Code `.claude/rules` · Cursor `.cursor/rules`）

`.claude/rules/*.md` 用 YAML frontmatter 的 `paths:` 字段限定加载范围——Claude Code 原生等价于 `.cursor/rules` 的 globs：

```yaml
---
paths:
  - "src/api/**"
  - "**/*.handler.ts"
---
所有 API handler 必须先用 Zod 校验输入。
```

无 `paths` 的 rule 等同 CLAUDE.md（常驻、耗 token），应避免。横切多个（但非全部）目录的约束（如“migrations 是 append-only”）适合 path-scoped rule；仅子目录专属的约定适合子目录 CLAUDE.md。

---

## Output styles 与 append-system-prompt（慎用 · Claude Code 专属）

- **Output styles**（`.claude/output-styles/`）注入系统提示，**永不压缩、加载权重最高**——但**会覆盖默认系统提示**（除非 frontmatter 设 `keep-coding-instructions: true`）。
  误用会让 Claude 从“软件工程师助手”退化为“通用助手”，丢失变更范围控制、注释时机、验证习惯等默认指令。优先用内置的 `Proactive` / `Explanatory` / `Learning`。
- **append-system-prompt**（`--append-system-prompt` CLI flag）是**追加**而非替换，仅当次调用生效，更安全。适合领域知识、格式偏好。但收益递减——指令越多遵从越松，矛盾指令尤甚。

---

## Plugins（打包分发 · Claude Code 专属）

[Plugins](https://code.claude.com/docs/en/plugins) 把 skills + agents + hooks + MCP servers + output styles 打包成可分发单元（`plugin.json` + 组件目录）。团队级一致环境（一套 hooks + 一套 skills + 共享 MCP）适合做成 plugin 跨项目复用。

---

## 与本仓库原则的映射

| 仓库原则 | 机制层落地 |
|---|---|
| `writing-agent-docs` A.2“确定性约束优先”| 能用 `PreToolUse` hook 阻断的，不写进 AGENTS.md |
| `writing-agent-docs` C.5“工作流闭包——完成标准”| `Stop` 的 prompt/agent hook 验证“真的完成”而非代理自报 |
| `structuring-project` 三层边界 `Never` | `Never` 类禁令 → hook + permissions（确定性兜底）|
| `structuring-project` Toolchain First | hook 是 Claude Code 原生工具链的一部分 |
| `writing-skill-md` 安全考虑 | `allowed-tools` 最小权限 + hook 收紧 = 纵深防御 |

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|----------|
| [R1] | <https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more> | Steering Claude Code | Anthropic 官方博客，介绍 Claude Code 的八种指令方法及其决策表 |
| [R2] | <https://code.claude.com/docs/en/workflows> | Orchestrate Subagents at Scale with Dynamic Workflows | Claude Code Dynamic Workflows 官方文档（GA，v2.1.154+）：编排脚本、保存命令、ultracode 模式 |
| [R3] | <https://code.claude.com/docs/en/hooks-guide> | Hooks Guide | Claude Code Hooks 完整使用指南 |
| [R4] | <https://code.claude.com/docs/en/hooks> | Hooks Reference | Claude Code Hooks API 参考 |
| [R5] | <https://code.claude.com/docs/en/hooks#hook-lifecycle> | Hook Lifecycle | Claude Code Hooks 生命周期事件完整表 |
| [R6] | <https://code.claude.com/docs/en/sub-agents> | Subagents | Claude Code Subagents 文档——隔离上下文执行 |
| [R7] | <https://code.claude.com/docs/en/plugins> | Plugins | Claude Code Plugins 文档——打包分发技能、代理、hooks、MCP server |
| [R8] | <https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code> | A Harness for Every Task | Anthropic 官方博客，DW 初始发布的模式介绍（fan-out/synthesize、/loop+/goal）|
