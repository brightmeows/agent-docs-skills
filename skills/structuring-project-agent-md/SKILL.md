---
name: structuring-project-agent-md
description: 指导项目级代理配置的创建与维护——层级结构、Toolchain First、Always/Ask/Never 边界、常见异味检测。在创建或修改 AGENTS.md、CLAUDE.md、.cursor/rules 等文件，或代理行为不符合预期时使用。
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

**大小参考**：无最低行数要求；建议 100–150 行，不超过 200 行（Anthropic 官方建议 <200 行；arXiv:2606.15828 对 100 个热门仓库分析发现 42% 的文件 >200 行并出现 Context Bloat）。
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
- **@import 引用**——部分工具（如 Claude Code）支持 `@路径/文件名.md` 内联引用外部文件，根文件保持精简，知识按需加载。非 v1.1 标准特性（进展见 [cross-tool-compat.md](reference/cross-tool-compat.md#标准化进展)），使用前确认工具兼容性。
- **重点标注非常规**——主流实践、常见配置等显而易见的内容一笔带过；非常规、反直觉、项目特有的内容重点提及。

### 决策表（Decision Tables）

当项目中存在两种或多种合理做法时，决策表能强制让代理在选择前做出决定，而不靠猜测。来自 [AugmentCode 实证研究](https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files) 的最强模式之一——将决策表加入 AGENTS.md 后 `best_practices` 提升 25%。

```markdown
## 状态管理选型

| 场景 | → React Query | → Zustand |
|------|:---:|:---:|
| 服务器是唯一数据源 | ✅ | |
| 多代码路径修改此状态 | | ✅ |
| 需要乐观更新 + 本地状态 | | ✅ |
```

**原则：**

- 用表格而非文字描述对比，代理可直接查表决策
- 列是选项，行是判断条件，单元格标记适用性
- 适用于框架选型、模式选择、架构决策等场景
- 避免超出 3–4 列，列过多时优先简化

**何时用约束 vs 决策表：** 两者互补而非互斥

| 场景 | 用整体约束 | 用决策表 |
|------|-----------|---------|
| 默认路径清晰，代理可从示例自行推断 | ✅ 简短的规则即可 | ❌ 冗余 |
| 存在多个合理选项，靠猜测选型可能出错 | ❌ 不够精确 | ✅ 强制选择 |
| 只有一两个判断维度 | ✅ | 可也可不用 |
| 判断维度多、需要综合对比 | ❌ 约束难以穷举 | ✅ 一目了然 |

- 先用约束覆盖基线场景，出现选型混乱时再补充决策表

### 维护规则

以下为 AGENTS.md 专属：

- **写前检查**：遍历锁文件、CI 和现有代码模式再落笔。
- **定期审计**：周期性审查后将可被工具链强制的规则迁移出去（指向工具配置）。

### 常见错误

| 错误 | 详见 |
|---|---|---|
| 含 README 内容 | 定位 |
| 重复工具链已强制内容（即 Lint Leakage）| 写作原则 Toolchain First / [配置异味检测](#配置异味检测) |
| 自动生成不审校（即 Init Fossilization）| 写作原则 反自动化生成 / [配置异味检测](#配置异味检测) |
| 不同工具各维护一份 | 维护唯一 AGENTS.md，symlink 到各工具入口文件（CLAUDE.md / GEMINI.md 等）|
| 否定指令 | 前置 Skill（肯定指令优先） |
| 边界规则混排在大表或单一列表 | 写作原则：三层边界 / 分小节放置 |
| CLAUDE.md 与 AGENTS.md 不相关而用 symlink | CLAUDE.md 专属指导 |
| 把个人偏好写在项目级配置 | writing-agent-docs → 作用域 |
| .cursor/rules 无 glob 范围 | .cursor/rules 格式节 |

### 配置异味检测

arXiv:2606.15828 分析了 100 个热门仓库的 AGENTS.md，识别出六种常见配置异味。编写和审计时应检查：

| 异味 | 说明 | 检查方法 |
|------|------|---------|
| **Lint Leakage**（最普遍，62%） | 重复 linter/formatter 已强制的规则（命名风格、缩进、import 排序） | 被 Biome/ESLint/Ruff 等工具能自动修复的 → 删除 |
| **Context Bloat**（42%） | 文件 >200 行，堆砌规则和细节 | 拆分到子目录 AGENTS.md 或技能文件 |
| **Skill Leakage**（35%） | 把仅特定场景需要的指令放入常驻 AGENTS.md（如测试指南、脚手架流程） | 迁移到技能或子文件按需加载 |
| **Conflicting Instructions**（28%） | 指令互相矛盾，如两个不同路径指向同一职责 | 定期审计，冲突处显式注明优先级 |
| **Init Fossilization**（24%） | 由 `/init` 自动生成后从未人工审校修改 | 创建后至少人工审校一次，迭代更新 |
| **Blind Reference**（16%） | 引用外部文件时不说明用途和场景 | 每条引用配一句话：什么内容、何时读 |

**核心原则**：配置越精炼、越聚焦项目特有内容，代理表现越好。冗余指令每多一条，关键规则的注意力就少一分。

> 异味 Lint Leakage 和 Init Fossilization 在上方[常见错误表](#常见错误)中也有对应条目（重复工具链已强制内容、自动生成不审校），从实践角度互为补充。

### 安全考虑

AGENTS.md 注入代理上下文，因此也引入安全风险。编写时注意：

- **Blind Reference 可能泄露敏感文件**——引用 `../internal/credentials.md` 等路径时，代理可能会读取并加载其内容。确保引用文件本身可安全暴露给代理。
- **不暴露内部基础设施细节**——无需在 AGENTS.md 中记录内网 IP、内部域名、未公开 API 端点等。代理不需要这些信息来完成任务。
- **`/init` 生成后必审校**——自动生成的配置文件常包含过于宽泛的架构描述和内部路径，发布前应人工清理。
- **凭证永不写入**——API token、数据库密码、SSH 密钥等即使注释掉也不应出现在 AGENTS.md 或引用文件中。

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
|---|---|
| [reference/content-decisions.md](reference/content-decisions.md) | 附录：内容决策详细目录（维度评分 / 放入条件 / 根子目录拆分） |
| [reference/comparison-tools.md](reference/comparison-tools.md) | AGENTS.md vs Skill vs MCP 对比（含 token 开销） |
| [reference/empirical-evidence.md](reference/empirical-evidence.md) | Princeton/ETH Zurich/上下文效率等实证数据 |
| [agent-persona.md](../writing-agent-docs/agent-persona.md) | Agent Persona 完整定义（项目级 + 个人级，跨技能共享） |
| [reference/auto-gen-warning.md](reference/auto-gen-warning.md) | LLM 自动生成危害与实证数据 |
| [reference/cross-tool-compat.md](reference/cross-tool-compat.md) | 跨工具概念对照 + AGENTS.md v1.1 标准化进展 |
| [reference/claude-md.md](reference/claude-md.md) | CLAUDE.md 专属指导（Symlink 策略、独有特性、Commands 目录）|
| [reference/cursor-rules.md](reference/cursor-rules.md) | .cursor/rules .mdc 格式（字段说明、与 AGENTS.md 职责划分）|
