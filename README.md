# Agent Docs Skills

> 为 AI 编码助手（OpenCode、Claude Code 等）提供的**关于编写代理文档本身的**元技能集合。
>
> 本仓库的每份技能都在编写过程中遵循了自身所教授的原则——是对技能开发方法论的践行而非仅说教。

为 AI 编码助手编写的代理文档（AGENTS.md、SKILL.md）需要遵循特定的写作方法才能有效引导代理行为。本仓库记录并形式化了这些方法。

## 技能

| 技能 | 前驱依赖 | 说明 |
|------|----------|------|
| [writing-agent-docs](skills/writing-agent-docs/SKILL.md) | — | **代理文档写作通用规则**。所有面向代理文本的基础（AGENTS.md、SKILL.md、.cursor/rules、系统提示词等）。核心原则：上下文是公共资源、每 token 须自证价值、确定性约束优先。 |
| [writing-skill-md](skills/writing-skill-md/SKILL.md) | writing-agent-docs | **SKILL.md 技能编写**。面向代理的实操指南：技能类型、目录结构、SKILL.md 结构、CSO、流程图、反模式、写完即自检。末尾含人类作者 TDD 验证方法参考（压力测试、对抗合理化）。 |
| [structuring-project-agent-md](skills/structuring-project-agent-md/SKILL.md) | writing-agent-docs | **项目级代理配置指南**（AGENTS.md / CLAUDE.md / .cursor/rules）。层级作用域、Always/Ask/Never 三层边界、Toolchain First、反自动化生成、增量迭代法。含 CLAUDE.md 专属指导与 .cursor/rules 格式。 |
| [structuring-personal-agent-md](skills/structuring-personal-agent-md/SKILL.md) | writing-agent-docs | **个人级代理配置指南**（CLAUDE_GLOBAL.md / 个人 AGENTS.md）。全局偏好、Persona 定义、个人级写作原则与常见模式。 |

### 依赖关系

```
writing-agent-docs（基础写作原则）
├── writing-skill-md（SKILL.md 格式——跨项目级/个人级）
├── structuring-project-agent-md（项目级配置：AGENTS.md / CLAUDE.md / .cursor/rules）
└── structuring-personal-agent-md（个人级配置：CLAUDE_GLOBAL.md / 个人 AGENTS.md）
```

## 技能间的引用约定

本仓库技能通过相对路径 `../<skill>/SKILL.md` 互相引用。激活上游技能是下游技能的前提条件——使用前请确保加载依赖链。

## 使用方式

### npx skills（推荐）

锁定到指定 release tag：

```bash
npx skills add https://codeberg.org/brightmeows/agent-docs-skills.git#v0.1.0
```

拉取 `main` 分支，始终最新：

```bash
npx skills add https://codeberg.org/brightmeows/agent-docs-skills/raw/branch/main
```

### 手动引用

克隆仓库后，在 AI 助手的配置中引用 `skills/` 下的 `SKILL.md`：

```bash
git clone https://codeberg.org/brightmeows/agent-docs-skills.git
```

## 贡献

欢迎提交 Issue 或 Pull Request。内容纠错、示例补充、反模式记录等都十分感谢。

请阅读 [AGENTS.md](AGENTS.md) 了解开发工作流。

## 许可

Apache-2.0
