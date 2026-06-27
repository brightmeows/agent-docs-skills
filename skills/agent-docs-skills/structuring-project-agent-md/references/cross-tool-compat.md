# 跨工具兼容性参考

> `structuring-project-agent-md` 参考文件。
> 不同工具对同一概念有不同的命名、路径和加载方式；写跨工具配置时参考此表。含 AGENTS.md v1.1 标准化进展。

---

## 概念对照

| 概念 | AGENTS.md 术语 | Claude Code | Cursor | OpenCode | Hermes Agent [R3] |
|------|---------------|-------------|--------|----------|-------------------|
| **项目级规则** | `./AGENTS.md` | `./CLAUDE.md` | `.cursor/rules/*.mdc` | `opencode.json` + `.opencode/` | —（个人级为主）|
| **个人级规则** | — | `~/.claude/CLAUDE_GLOBAL.md` | `~/.cursor/rules/`（全局）| `~/.config/opencode/AGENTS.md` | `~/.hermes/SOUL.md`（身份）|
| **技能存放（项目）** | `./.well-known/` | `.claude/skills/` | `.cursor/skills/` | `.opencode/skills/` | — |
| **技能存放（个人）** | — | `~/.claude/skills/` | `~/.cursor/skills/` | `~/.agents/skills/` | `~/.hermes/skills/` |
| **技能发现文件** | `.well-known/agent-skills/index.json` | 同左 | 同左 | 同左 | 同左 |
| **技能权限控制** | — | `allowed-tools` frontmatter | — | `permission.skill` | `write_approval` 门控 + 安全扫描器 |
| **工具配置（项目）** | — | `.claude/settings.json` | `.cursor/settings.json` | `opencode.json` | — |
| **工具配置（个人）** | — | `~/.claude/settings.json` | `~/.cursor/settings.json` | `~/.config/opencode/opencode.json` | `~/.hermes/config.yaml` |
| **角色定义** | `AGENTS.md` 中写 Persona | `CLAUDE.md` 或 skill | `.cursor/rules/` | `.opencode/agents/*.md` | `SOUL.md`（持久身份） |
| **Agent 定义** | — | `.claude/agents/*.md` subagent | — | `.opencode/agents/*.md` | 内置 agent 类型 |
| **子目录规则** | 子目录 `AGENTS.md` | 子目录 `CLAUDE.md` | `.cursor/rules/` 多文件 | 继承 `opencode.json` | — |
| **常驻配置格式** | Markdown | Markdown | Markdown + YAML frontmatter | JSON / JSONC + Markdown | Markdown + YAML + YAML bundles |
| **特有机制** | — | Hooks / Subagents / DW | Composer / Agent 模式 | Task agent / 多 provider | `/learn` 自动创建技能 / skill bundles / 自演进技能 |

---

## 文件名对照

下表列出来主要工具的配置文件及与 AGENTS.md 的关系。多数工具近年新增的配置（如 Factory、Amp、Goose、Augment Code、Devin）均直接复用 `AGENTS.md` 作为项目级指令文件——这本身就是 AGENTS.md 标准化效果的体现。如使用未列出的新工具，先检查其是否原生支持 `AGENTS.md`。

| 工具 | 文件 | 与 AGENTS.md 关系 |
|------|------|-------------------|
| **AGENTS.md 原生** | `AGENTS.md` | 直接读取，跨工具标准（AAIF 维护） |
| **Claude Code** | `CLAUDE.md` | 工具原生格式；可 symlink 到 AGENTS.md 共享真理源 |
| **Gemini CLI** | `GEMINI.md` | 工具原生格式；可配置指向 AGENTS.md |
| **GitHub Copilot** | `.github/copilot-instructions.md` | 原生指令文件；亦读 AGENTS.md（[R1] 起 code review 支持）|
| **Windsurf** | `.windsurfrules` / `.windsurf/rules/*.md` | 原生规则文件；亦读 AGENTS.md 作为 fallback |
| **Cursor** | `.cursor/rules/*.mdc` | 按 glob 匹配注入；与 AGENTS.md 职责互补 |
| **JetBrains Junie** | `.junie/guidelines.md` | 原生格式 |
| **OpenCode** | `opencode.json` | 项目级工具配置（含规则路径引用）|
| **Hermes Agent** ([R3]) | `~/.hermes/skills/` | 个人级技能存放 + `SOUL.md` 持久身份 |
| **Aider** | `.aider.conf.yml` | 通过配置指向 AGENTS.md |

---

## 加载差异

各工具的完整加载流水线与个人级 / 项目级优先级见 [scope-and-loader.md](../../writing-agent-docs/references/scope-and-loader.md#典型工具的加载差异)。

---

## 标准化进展

### 治理归属：AAIF / Linux Foundation

AGENTS.md **已经正式归入 Agentic AI Foundation（AAIF）**（[R4]），
后者是 Linux Foundation 下属的专项基金。AAIF 还托管了 MCP
（Model Context Protocol）和 Goose（Block 捐赠的开源 agent），
为 agent 生态提供中立治理框架。AGENTS.md 的主页
（[agents.md](https://agents.md)）及规范均在 AAIF 下维护。

### v1.1 提案状态

AGENTS.md v1.1 处于**草案提案**阶段（[R2]，尚未合入，保持完全向后兼容）。提案明确了管辖范围、累积、优先级、隐式继承四大语义，并定义了 AGENTS.md 与 SKILL.md 的职责边界（behavior vs capabilities）。

**YAML Frontmatter**（渐进式披露，提案为可选）：可选的 frontmatter 允许代理在加载全文前建立轻量索引。`description` 和 `tags` 均为可选——文件路径本身已提供足够上下文，不要求 frontmatter 以保持向后兼容。

```yaml
---
description: React frontend conventions and build commands
tags: [react, frontend, ui]
---
```

frontmatter 帮助代理判断何时需要加载该文件的完整内容，无需全文扫描。

---

### 相关规范：Google ARD（Agentic Resource Discovery）

Google 联合多家合作伙伴发布 **Agentic Resource Discovery（ARD）** 开放规范（[R5]），
定义了 agent 资源（技能、工具、agent 自身）的发布、发现与验证协议。

ARD 与 `.well-known/agent-skills/index.json` 的异同：

| 维度 | `.well-known/index.json`（agentskills.io） | ARD（agenticresourcediscovery.org） |
|------|------------------------------------------|-------------------------------------|
| **定位** | 技能索引清单，位于仓库根目录 | 发现协议，可托管于任意 Web 服务器 |
| **发现方式** | 约定路径 `.well-known/agent-skills/` | HTTP `Link` header / `/.well-known/ard` |
| **覆盖范围** | 仅 Agent Skills | Skills + MCP Servers + Agents |
| **验证机制** | digest SHA256 匹配 | 签名 + 元数据校验 |
| **两者关系** | 互补——`.well-known/` 管理技能索引，ARD 扩展发现范围到全类 agent 资源 | |

写跨工具配置时先确保 `.well-known/agent-skills/index.json` 有效（所有市场均支持），
需要跨类资源发现时才考虑 ARD。

---

## 迁移指南

### 从单一 AGENTS.md 扩展到多工具

1. 在 `AGENTS.md` 中维护跨工具共享内容
2. 如果 Claude Code 需要独有指令 → 建 symlink `CLAUDE.md → AGENTS.md` 或独立文件
3. 如果 Cursor 需要文件级规则 → `.cursor/rules/*.mdc` 配 globs
4. 检查工具兼容性：`@import`、`Commands` 等特性可能不跨工具

### <a name="converge-to-single-truth"></a>从多份独立文件收敛到单一真理源

1. 对比各工具配置文件，找出重叠内容
2. 重叠内容写入 `AGENTS.md`
3. 独有内容留在各工具原生配置中
4. 用 symlink 或工具配置指向 AGENTS.md

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|----------|
| [R1] | <https://github.blog/changelog/2026-06-18-copilot-code-review-agents-md-support-and-ui-improvements/> | Copilot code review AGENTS.md support and UI improvements | GitHub Copilot 在 code review 中支持 AGENTS.md |
| [R2] | <https://github.com/agentsmd/agents.md/issues/135> | AGENTS.md v1.1 proposal | AGENTS.md v1.1 草案，明确管辖范围、累积、优先级、隐式继承四大语义 |
| [R3] | <https://hermes-agent.nousresearch.com/docs/user-guide/features/skills> | Hermes Agent Skills System | 开源 self-improving agent；`/learn` 自动创建 SKILL.md、三级渐进披露、skill bundles、`SOUL.md` 持久记忆 |
| [R4] | <https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation> | Linux Foundation Announces the Formation of the AAIF | AAIF 成立公告，AGENTS.md 归入 Linux Foundation 旗下 |
| [R5] | <https://developers.googleblog.com/announcing-the-agentic-resource-discovery-specification/> | Announcing the Agentic Resource Discovery Specification | Google ARD：agent 资源（技能/工具/agent）发布、发现与验证的开放规范 |
