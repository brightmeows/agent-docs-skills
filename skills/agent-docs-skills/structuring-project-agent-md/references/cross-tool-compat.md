# 跨工具兼容性参考

> `structuring-project-agent-md` 参考文件。
> 不同工具对同一概念有不同的命名、路径和加载方式；写跨工具配置时参考此表。含 AGENTS.md v1.1 标准化进展。

---

## 概念对照

| 概念 | AGENTS.md 术语 | Claude Code | Cursor | OpenCode |
|------|---------------|-------------|--------|----------|
| **项目级规则** | `./AGENTS.md` | `./CLAUDE.md` | `.cursor/rules/*.mdc` | `opencode.json` |
| **个人级规则** | — | `~/.claude/CLAUDE_GLOBAL.md` | `~/.cursor/rules/`（全局）| `~/.config/opencode/` |
| **技能存放（项目）** | `./.well-known/` | `.claude/skills/` | `.cursor/skills/` | `.opencode/skills/` |
| **技能存放（个人）** | — | `~/.claude/skills/` | `~/.cursor/skills/` | `~/.agents/skills/` |
| **技能发现文件** | `.well-known/agent-skills/index.json` | 同左 | 同左 | 同左 |
| **工具配置（项目）** | — | — | — | `opencode.json` / `.opencode/` |
| **工具配置（个人）** | — | `~/.claude/settings.json` | `~/.cursor/settings.json` | `~/.config/opencode/opencode.json` |
| **角色定义** | `AGENTS.md` 中写 Persona | `CLAUDE.md` 或 skill | `.cursor/rules/` | agent 文件或 inline config |
| **子目录规则** | 子目录 `AGENTS.md` | 子目录 `CLAUDE.md` | `.cursor/rules/` 多文件 | 继承 `opencode.json` |
| **常驻配置格式** | Markdown | Markdown | Markdown + YAML frontmatter | JSON / JSONC |

---

## 文件名对照

| 工具 | 文件 | 说明 |
|------|------|------|
| **Claude Code** | `CLAUDE.md` | 项目级，自动发现 |
| **Cursor** | `.cursor/rules/*.mdc` | 项目级，按 glob 匹配注入 |
| **Gemini CLI** | `GEMINI.md` | 项目级，自动发现 |
| **GitHub Copilot** | `.github/copilot-instructions.md` | 项目级原生指令文件；亦读 AGENTS.md（[2026-06-18](https://github.blog/changelog/2026-06-18-copilot-code-review-agents-md-support-and-ui-improvements/) 起 code review 支持）|
| **Windsurf** | `.windsurfrules` / `.windsurf/rules/*.md` | 项目级规则；亦读 AGENTS.md 作为 fallback |
| **JetBrains Junie** | `.junie/guidelines.md` | 项目级 |
| **OpenCode** | `opencode.json` | 项目级工具配置（含规则路径引用）|
| **通用（跨工具）** | `AGENTS.md` | 跨工具 fallback 标准 |

---

## 加载差异

各工具的完整加载流水线与个人级 / 项目级优先级见 [scope-and-loader.md](../../writing-agent-docs/references/scope-and-loader.md#典型工具的加载差异)。

---

## 标准化进展

AGENTS.md v1.1 处于**草案提案**阶段（[GitHub issue #135](https://github.com/agentsmd/agents.md/issues/135)，尚未合入，保持完全向后兼容）。提案明确了管辖范围、累积、优先级、隐式继承四大语义，并定义了 AGENTS.md 与 SKILL.md 的职责边界（behavior vs capabilities）。

**YAML Frontmatter**（渐进式披露，提案为可选）：可选的 frontmatter 允许代理在加载全文前建立轻量索引。`description` 和 `tags` 均为可选——文件路径本身已提供足够上下文，不要求 frontmatter 以保持向后兼容。

```yaml
---
description: React frontend conventions and build commands
tags: [react, frontend, ui]
---
```

frontmatter 帮助代理判断何时需要加载该文件的完整内容，无需全文扫描。

---

## 迁移指南

### 从单一 AGENTS.md 扩展到多工具

1. 在 `AGENTS.md` 中维护跨工具共享内容
2. 如果 Claude Code 需要独有指令 → 建 symlink `CLAUDE.md → AGENTS.md` 或独立文件
3. 如果 Cursor 需要文件级规则 → `.cursor/rules/*.mdc` 配 globs
4. 检查工具兼容性：`@import`、`Commands` 等特性可能不跨工具

### 从多份独立文件收敛到单一真理源

1. 对比各工具配置文件，找出重叠内容
2. 重叠内容写入 `AGENTS.md`
3. 独有内容留在各工具原生配置中
4. 用 symlink 或工具配置指向 AGENTS.md
