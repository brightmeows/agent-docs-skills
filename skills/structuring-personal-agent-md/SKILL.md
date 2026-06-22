---
name: structuring-personal-agent-md
description: 指导个人级代理配置的创建——个人偏好表达、Persona 设定、常驻上下文精简。在创建或编辑 ~/.claude/CLAUDE_GLOBAL.md、个人 AGENTS.md、~/.cursor/rules 全局规则时使用。
license: Apache-2.0
---

# 个人级代理配置技能

本技能覆盖 home 目录中个人版 AGENTS.md / CLAUDE.md / `.cursor/rules` 等面向代理的配置文件。

---

## 前置 Skill

**必须先激活 [`writing-agent-docs`](../writing-agent-docs/SKILL.md)。**

该技能定义代理文档写作的通用规则。本技能仅承载个人级配置专属内容，不重复通用规则——遇通用写作决策时回退到前置 Skill。

> **作用域**：本技能仅覆盖**个人级**（home 目录）代理配置文件。项目级同类配置见 [`structuring-project-agent-md`](../structuring-project-agent-md/SKILL.md)。

---

## 定位

个人级代理配置文件告诉 agent**你希望它如何为你工作**，独立于任何项目。

与项目级配置的区别（位置 / 作用域 / 谁写 / 版本控制 / 生命周期 / 内容 / 优先级）见 [scope-and-loader.md 的“两类作用域对比”](../writing-agent-docs/scope-and-loader.md#两类作用域对比)。

---

## 配置文件位置

完整的个人级配置文件清单、加载顺序、与项目级的优先级规则见 [scope-and-loader.md](../writing-agent-docs/scope-and-loader.md)。写作时最常用：Claude Code 的 `~/.claude/CLAUDE_GLOBAL.md`、通用的 `~/.agents/AGENTS.md`、OpenCode 的 `~/.config/opencode/AGENTS.md`。

**加载优先级**：就近优先——个人级定义通用行为基调，项目级在冲突时覆盖（详见 [scope-and-loader.md](../writing-agent-docs/scope-and-loader.md#优先级规则)）。

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

> 个人级 Persona → 项目级 Persona → 技能级 Persona，按此顺序**累积**。项目级覆盖个人级冲突部分。详见 [agent-persona.md](../writing-agent-docs/agent-persona.md)。

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
```

---

## 常见错误

| 错误 | 正确做法 |
|------|---------|
| 把项目特有命令写入个人级 | 仅写通用偏好，项目特有放 AGENTS.md |
| 个人级配置过长（>30 行）| 精简到 5–15 行，长内容用技能替代 |
| 用否定指令表达偏好 | 改写为肯定替代（见 writing-agent-docs） |
| 在个人级重复项目级已覆盖的内容 | 信任就近优先规则——个人级只写个人偏好 |
| 把 SKILL.md 当个人级配置 | SKILL.md 是按需加载的任务知识，个人级是常驻偏好 |
