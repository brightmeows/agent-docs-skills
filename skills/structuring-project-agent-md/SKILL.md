---
name: structuring-project-agent-md
description: 在创建、修改或重构项目级代理配置文件（AGENTS.md、CLAUDE.md、.cursor/rules、GEMINI.md、.junie/guidelines.md 等）时使用。代理行为不符合预期时亦适用。
license: Apache-2.0
---

# 项目级代理配置技能

本技能覆盖项目中**所有**面向代理的配置文件（AGENTS.md、CLAUDE.md、.cursor/rules 等）的创建与维护。

---

## 前置 Skill

**必须先激活 [`writing-agent-docs`](../writing-agent-docs/SKILL.md)。**

该技能定义代理文档写作的通用规则。本技能仅承载项目级配置专属内容，不重复通用规则——遇通用写作决策时回退到前置 Skill。

> **作用域**：本技能仅覆盖**项目级**（仓库内）代理配置文件。个人级（home 目录）的同类配置见 [`structuring-personal-agent-md`](../structuring-personal-agent-md/SKILL.md)。

---

## 定位

项目级配置文件告诉代理**在此项目中如何工作**。各文件的定位：

| 文件 / 目录 | 定位 | 工具原生支持 |
|-------------|------|-------------|
| `AGENTS.md` | 跨工具标准，项目约定 / 命令 / 边界 | 60,000+ 仓库、30+ 工具 |
| `CLAUDE.md` | Claude Code 原生项目配置 | Claude Code |
| `.cursor/rules/*.mdc` | Cursor 文件匹配规则 | Cursor、OpenCode 等 |
| `GEMINI.md` | Gemini CLI 项目配置 | Gemini CLI |
| `.junie/guidelines.md` | JetBrains Junie 配置 | JetBrains Junie |

与 README 职责分离（README 面向人，项目级配置面向代理）。

**核心原则**：维护一个主要的 AGENTS.md 作为跨工具真理源，通过 symlink 或工具配置让各工具读取。仅在跨工具有实质性行为差异时维护独立文件。

---

## 层级与作用域

AGENTS.md 按文件系统层级组织，遵循 4 核心作用域概念：

| 概念 | 含义 |
|---|---|
| **管辖范围** | 每个 AGENTS.md 只影响所在目录及子目录，不影响兄弟目录 |
| **累积** | 子目录继承祖先文件的全部指导，无需重复声明 |
| **优先级** | 就近优先，子目录覆盖祖先的冲突规则 |
| **隐式继承** | 子文件免重复祖先规则——代理视指导为累积的 |

| 层级 | 内容范围 |
|---|---|
| **根 AGENTS.md** | 项目描述、共享工具链、全局边界、代码风格基础 |
| **子目录 AGENTS.md** | 该包领域逻辑、局部技术栈、包内特有命令和约定 |
| **深层 AGENTS.md** | 极端特化规则，覆盖祖先的不适用约束 |

根文件应精简，内容冗余则下沉到子目录。子文件只声明该目录特有内容，祖先已声明的无需重复。

---

## CLAUDE.md 专属指导

CLAUDE.md 是 Claude Code 原生读取的项目级配置文件。核心策略是 symlink 到 AGENTS.md 以维护单一真理源；当需要使用 `@import`、Commands 等 Claude Code 独有特性时，可维护独立文件。

详细策略、独有特性、Commands 目录说明见 [`reference/claude-md.md`](reference/claude-md.md)。

---

## .cursor/rules 格式

`.cursor/rules/` 目录使用 `.mdc` 文件格式，通过 `globs` 字段按文件匹配注入规则（`alwaysApply: true` 则常驻）。与 AGENTS.md 的职责划分：AGENTS.md 管全局约定，`.mdc` 管文件级规则。

详细格式、字段说明、职责对比、选用指南见 [`reference/cursor-rules.md`](reference/cursor-rules.md)。

---

## 创建与维护流程

### 增量迭代法

1. **起步**：仅覆盖最常出错的命令和边界，从简开始
2. **观察**：用真实任务记录代理反复出错的地方
3. **补充**：将反复出现的问题写入
4. **精简**：代理已能遵循的规则可移除
5. **重复**

**大小参考**：无最低行数要求；最高不建议超过 150 行（参考 [CLAUDE.md best practices](https://automationswitch.com/ai-workflows/skillmd-vs-agentsmd-vs-claudemd-when-to-use-each)）。
超出后冗余内容会推高推理成本、削弱代理对关键规则的注意力（Gloaguen et al., 2026：context file 中不必要指令使推理 token 增加 14–22%——详见 [empirical-evidence.md](reference/empirical-evidence.md)）。

### 重构

- **审计**：遍历现有指令，标记可由工具链强制或已过时的内容
- **拆分**：按层级将臃肿根文件内容分配到子目录
- **迁移**：将确定性规则移出 AGENTS.md，指向工具配置
- **验证**：改动前后对比，确认代理推理开销未增加

### 写作原则

以下为 AGENTS.md 专属：

- **Toolchain First**——确定性约束（代码风格、类型、构建、测试）归属工具链配置，AGENTS.md 只承载建议性指令（架构判断、工作流偏好）。

  ```
  # 好——指向工具，不重复规则
  Lint: `pnpm lint`（Biome——见 biome.json）
  # 坏——代替工具写规则
  不要用 var，始终用 const/let，import 顺序按标准库/三方/内部排列...
  ```

- **三层边界 Always / Ask / Never**——比简单禁令清单更有效：
  - **Always Do**：每次自动执行（如提交前运行 `pnpm test`）
  - **Ask First**：重大变更先确认（如改数据库 schema）
  - **Never Do**：绝对禁止（如提交密钥、push main）——配肯定替代（见前置 Skill“肯定指令优先”）
  - **分小节放置**：用 `### Always` / `### Ask` / `### Never` 独立小节分类承载各条目，代理跳读时可快速定位。混排在大表或单一列表中会削弱分类索引价值。
- **反自动化生成**——LLM 自动生成的 AGENTS.md 一致降低成功率、推高推理成本（完整数据与机制见 [reference/auto-gen-warning.md](reference/auto-gen-warning.md)）。`/init` 等结果只当“内容清单”，手工重写。
- **关键文件路径显式标注**——入口点、基类、配置文件应显式标注路径。
- **@import 引用**——部分工具（如 Claude Code）支持 `@路径/文件名.md` 内联引用外部文件，根文件保持精简，知识按需加载。非 v1.1 标准特性，使用前确认工具兼容性。
- **重点标注非常规**——主流实践、常见配置等显而易见的内容一笔带过；非常规、反直觉、项目特有的内容重点提及。

### 维护规则

以下为 AGENTS.md 专属：

- **写前检查**：遍历锁文件、CI 和现有代码模式再落笔。
- **定期审计**：周期性审查后将可被工具链强制的规则迁移出去（指向工具配置）。

### 常见错误

| 错误 | 详见 |
|---|---|---|
| 含 README 内容 | 定位 |
| 重复工具链已强制内容 | 写作原则 Toolchain First |
| 自动生成不审校 | 写作原则 反自动化生成 / [auto-gen-warning.md](reference/auto-gen-warning.md) |
| 不同工具各维护一份 | 维护唯一 AGENTS.md，symlink 到各工具入口文件（CLAUDE.md / GEMINI.md 等）|
| 否定指令 | 前置 Skill（肯定指令优先） |
| 边界规则混排在大表或单一列表 | 写作原则：三层边界 / 分小节放置 |
| CLAUDE.md 与 AGENTS.md 不相关而用 symlink | CLAUDE.md 专属指导 |
| 把个人偏好写在项目级配置 | writing-agent-docs → 作用域 |
| .cursor/rules 无 glob 范围 | .cursor/rules 格式节 |

---

## 内容决策指南

什么内容该放入 AGENTS.md、什么不该放，按 5 维度评分，越高越该放入：

| 维度 | 含义 |
|------|------|
| **重要性** | 对代理理解项目的重要程度 |
| **推断难度** | 从项目自身获取的困难程度 |
| **稳定性** | 内容变更频率，越稳定越该放 |
| **特异性** | 项目特有程度，越特有越该放 |
| **可工具化程度** | 一票否决——可被工具链强制的内容不放 |

**判定**：重要性高 + 推断难 + 特异 → 放入；可工具化（linter / 类型 / CI 能强制）→ 不放，指向工具配置；介于之间 → 视项目复杂度 / 团队 / 安全要求酌情放入。

各内容类型的分类（推荐 / 可选 / 不放）、详细维度评分与根 / 子目录拆分见 [reference/content-decisions.md](reference/content-decisions.md)（附录：详细目录）。

---

## 参考文件

| 文件 | 内容 |
|---|---|---|
| [reference/content-decisions.md](reference/content-decisions.md) | 附录：内容决策详细目录（维度评分 / 放入条件 / 根子目录拆分） |
| [reference/comparison-tools.md](reference/comparison-tools.md) | AGENTS.md vs Skill vs MCP 对比（含 token 开销） |
| [reference/empirical-evidence.md](reference/empirical-evidence.md) | Princeton/ETH Zurich/上下文效率等实证数据 |
| [reference/agent-persona.md](reference/agent-persona.md) | Agent Persona 完整定义（项目级 + 个人级） |
| [reference/auto-gen-warning.md](reference/auto-gen-warning.md) | LLM 自动生成危害与实证数据 |
| [reference/v1.1-features.md](reference/v1.1-features.md) | AGENTS.md v1.1 YAML Frontmatter |
| [reference/cross-tool-compat.md](reference/cross-tool-compat.md) | AGENTS.md / CLAUDE.md / Cursor / OpenCode 概念对照 |
| [reference/claude-md.md](reference/claude-md.md) | CLAUDE.md 专属指导（Symlink 策略、独有特性、Commands 目录）|
| [reference/cursor-rules.md](reference/cursor-rules.md) | .cursor/rules .mdc 格式（字段说明、与 AGENTS.md 职责划分）|
