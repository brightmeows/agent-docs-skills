---
name: structuring-personal-agent-md
description: 指导个人级代理配置文件的创建与精简。在需要跨所有项目统一 agent 偏好、设置 Persona，或编辑 ~/.claude/CLAUDE_GLOBAL.md、~/.agents/AGENTS.md 等个人配置、个人配置超过 15 行时使用。
license: Apache-2.0
---

# 个人级代理配置技能

本技能覆盖 home 目录中个人版 AGENTS.md / CLAUDE.md / `.cursor/rules` 等面向代理的配置文件。

---

## 前置 Skill

**必须先激活 [`writing-agent-docs`](../writing-agent-docs/SKILL.md)——禁止以任何理由绕过此步骤。**

该技能定义代理文档写作的通用规则。本技能仅承载个人级配置专属内容，不重复通用规则——遇通用写作决策时回退到前置 Skill。

> **作用域**：本技能仅覆盖**个人级**（home 目录）代理配置文件。项目级同类配置见 [`structuring-project-agent-md`](../structuring-project-agent-md/SKILL.md)。

---

## 定位

个人级代理配置文件告诉 agent**你希望它如何为你工作**，独立于任何项目。

与项目级配置的区别（位置 / 作用域 / 谁写 / 版本控制 / 生命周期 / 内容 / 优先级）见 [L1] 的“两类作用域对比”。

---

## 配置文件位置

完整的个人级配置文件清单、加载顺序、与项目级的优先级规则见 [L1]。写作时最常用：

| 工具 | 个人级配置 | 用途 |
|------|-----------|------|
| Claude Code | `~/.claude/CLAUDE_GLOBAL.md` | 全局行为指令 |
| 通用 | `~/.agents/AGENTS.md` | 个人级 AGENTS.md |
| OpenCode | `~/.config/opencode/AGENTS.md` | 个人级全局规则 |
| OpenCode | `~/.config/opencode/opencode.json` | 工具配置 + agent 定义 |
| OpenCode | `~/.config/opencode/agents/*.md` | 自定义 agent 定义（Markdown agent 文件）|
| Cursor | `~/.cursor/rules/` | 全局规则文件 |
| Hermes Agent | `~/.hermes/SOUL.md` | 持久 agent 身份（Persona + 行为指令）|
| OpenClaw / Starpod | `SOUL.md` | 持久 agent 身份（姓名、角色、核心指令）|

**SOUL.md 模式**：OpenClaw、Hermes Agent、Starpod 等多个 agent 框架引入 `SOUL.md`
文件作为**持久 agent 身份**载体。与 `AGENTS.md` 或 `CLAUDE_GLOBAL.md` 不同，
`SOUL.md` 专门存放 agent 的身份特征（姓名、角色描述、核心行为原则），
被视为不可轻易覆盖的“身份层”——会话压缩后仍保留，而非每次重新注入。
个人级 Persona 可直接写在 `SOUL.md` 中，与项目级 `AGENTS.md` 形成三层身份叠加
（SOUL.md → AGENTS.md → SKILL.md HARD GATE）。此模式目前为 OpenClaw 生态专有，
但其“持久身份与项目配置分离”的设计理念值得借鉴。

**OpenCode agent 定义**：OpenCode 支持通过 Markdown agent 文件
（`~/.config/opencode/agents/*.md`）定义 agent 角色，含 YAML frontmatter
（description/mode/model/permission 等）和系统提示 body。也可在 `opencode.json` 中以
`agent` 字段配置。个人级 Persona 可直接在 agent 定义中设置。

**OpenCode** 支持 skill discovery 和 file-based agent 加载，
项目级 `AGENTS.md` 和 `SKILL.md` 自动被索引。个人级技能依然支持
`~/.agents/skills/` 目录。

**加载优先级**：就近优先——个人级定义通用行为基调，项目级在冲突时覆盖（详见 [L1] 的“优先级规则”）。

---

## 写作原则（个人级专属）

### 1. 专注于持续性，而非项目特异性

个人级配置跨所有项目生效，不要写入：

```markdown
# 坏：项目特有
本项目使用 pnpm，运行 pnpm dev 启动

# 好：通用偏好
我偏好 pnpm 而非 npm，新建项目时请用 pnpm init
```

### 2. 表达偏好，而非强硬规则

个人级配置表达“你希望怎么做”，项目级配置表达“这个项目要求怎么做”：

```markdown
# 坏：强硬规则
必须在提交前运行测试

# 好：个人偏好
我习惯在提交前先跑测试；如果项目有测试命令，建议先运行
```

### 3. 保持简短

个人级配置是常驻上下文的，越长对项目级有效内容的挤压越多。建议 **5–15 行**。

```markdown
# 推荐的个人 CLAUDE_GLOBAL.md
你是一个 senior 全栈工程师。
我的偏好：
- TypeScript + React + Tailwind
- 测试优先，但不强求 100% 覆盖
- 小提交、描述性 commit message
- 不喜欢死代码和注释掉的代码块
```

### 4. 可以在个人级定义 Persona

这是个人级配置最重要的用途——为 agent 设定角色：

```markdown
# ~/.claude/CLAUDE_GLOBAL.md
你是一个有 10 年经验的 Rust 后端工程师。
- 安全性和正确性优先于性能
- 显式错误处理，不用 unwrap/expect
- 文档注释（///）必须有语义价值
```

> 个人级 Persona → 项目级 Persona → 技能级 Persona，按此顺序**累积**。项目级覆盖个人级冲突部分。详见 [L2]。

---

## 常见模式

以下模式可按需取舍，组合成 5–15 行的个人配置：

```markdown
# 语言/工具链偏好
我主要用 TypeScript 和 Rust；新建 TS 项目用 pnpm + vitest，Rust 用 cargo nextest。
终端 zsh + starship；编辑器问题先查 .editorconfig 和 .vscode/。

# 安全基线（绝对）
永不将 API key/token 写入代码或提交；密钥用 `pass` 或 1password CLI 管理。

# 提交习惯
用 Conventional Commits（feat/fix/chore/docs/refactor）；提交前看 git diff --stat。

# 持久身份（SOUL.md 风格）
我叫 Alice，是一名资深全栈工程师。我偏好简洁、类型安全的代码，不写注释掉的死代码。
响应时直接给结论再解释，不要兜圈子。
```

### SOUL.md 持久身份模式

部分 agent 框架（OpenClaw、Hermes Agent、Starpod）支持 `SOUL.md` 文件定义**持久 agent 身份**。与常驻的个人级配置不同，`SOUL.md` 是「身份层」——定义 agent 是谁、说话风格、核心原则，会话压缩后仍保留。

```markdown
# ~/.hermes/SOUL.md（Hermes Agent 示例）
你是一位资深 Rust 工程师。你以精确、简洁著称，在架构讨论中优先考虑正确性。
- 回复直接、专业，偶尔带技术幽默
- 优先用类型系统表达约束，而非运行时检查
- 在安全性和性能之间始终选安全性
```

> `SOUL.md` 的写入和读取由框架管理，无需手动维护。此模式与技能安全中的记忆投毒风险相关——详见 [`writing-skill-md` 安全参考](../writing-skill-md/references/security.md)。

---

## 常见错误

| 错误 | 正确做法 |
|------|---------|
| 把项目特有命令写入个人级 | 仅写通用偏好，项目特有放 AGENTS.md |
| 用否定指令表达偏好 | 改写为肯定替代（见 writing-agent-docs） |
| 把 SKILL.md 当个人级配置 | SKILL.md 是按需加载的任务知识，个人级是常驻偏好 |

---

## 维护

个人级配置是常驻上下文，和项目级一样会膨胀。定期精简与初始写作同等重要——增量迭代的通用循环（起步→观察→补充→精简→重复）见前置 Skill A.1。

**膨胀信号**：

- 配置超过 15 行 → 挤压项目级有效内容的 token 预算
- 每次手动纠正代理后就加一条 → 配置在记录历史而非定义偏好
- 出现项目特有命令 → 错置，应移到项目级 AGENTS.md

**精简触发点**：

- **项目级已覆盖的删除**——信任就近优先，个人级只保留所有项目通用的偏好
- **代理已稳定遵循的删除**——反复纠正后已学会的偏好无需再写

---

## 本地参考

本技能引用的基础概念定义（位于前置 skill `writing-agent-docs`）：

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [scope-and-loader.md](../writing-agent-docs/references/scope-and-loader.md) | 作用域与加载顺序的唯一定义（文件清单、流水线、优先级、两类作用域对比）|
| [L2] | [agent-persona.md](../writing-agent-docs/references/agent-persona.md) | Agent Persona 定义（项目级 + 个人级，跨技能共享）|
