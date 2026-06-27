---
name: structuring-project-agent-md
description: 指导项目级代理配置文件的创建、维护与审计。在创建或修改仓库内 AGENTS.md、CLAUDE.md、.cursor/rules，或代理反复犯错、忽略项目约定、配置文件出现 Lint Leakage / Context Bloat / Skill Leakage 等配置异味时使用。
license: Apache-2.0
---

# 项目级代理配置技能

本技能覆盖项目中**所有**面向代理的配置文件（AGENTS.md、CLAUDE.md、.cursor/rules 等）的创建与维护。

---

## 前置 Skill

**必须先激活 [`writing-agent-docs`](../writing-agent-docs/SKILL.md)——禁止以任何理由绕过此步骤。**

该技能定义代理文档写作的通用规则。本技能仅承载项目级配置专属内容，不重复通用规则——遇通用写作决策时回退到前置 Skill。

> **作用域**：本技能仅覆盖**项目级**（仓库内）代理配置文件。个人级（home 目录）的同类配置见 [`structuring-personal-agent-md`](../structuring-personal-agent-md/SKILL.md)。

---

## 定位

项目级配置文件告诉代理**在此项目中如何工作**。各文件的定位：

| 文件 / 目录 | 定位 | 工具原生支持 |
|-------------|------|-------------|
| `AGENTS.md` | 跨工具标准，项目约定 / 命令 / 边界 | 事实标准——广泛采用 |
| `.well-known/agent-skills/index.json` | 技能发现清单（AAIF 标准） | 所有 SKILL.md 兼容工具 |
| `AGENTS.md` + ARD（[L5]） | 全类 agent 资源发现（技能/工具/agent）| Google 等联合发布的开放规范 |
| `CLAUDE.md` / `GEMINI.md` / 等 | 各工具原生项目配置（独有特性见 [L5]、[L6]） | 对应工具（多数亦读 AGENTS.md）|
| `.cursor/rules/*.mdc` | Cursor 文件匹配规则，按 glob 注入 | Cursor、OpenCode 等 |
| `opencode.json` | OpenCode 项目配置（工具/权限/agent 定义）| OpenCode |

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

根文件应精简；内容冗余时先归入当前文件的其它相关章节，无合适归处再下沉子目录。子文件只声明该目录特有内容，祖先已声明的无需重复。

### 渐进式披露：AGENTS.md 可选 Frontmatter

AGENTS.md v1.1（[R6]）草案提案引入了可选的 YAML frontmatter，支持代理在加载全文前建立轻量索引，适用于 monorepo 中含大量 AGENTS.md 文件的场景。`description` 和 `tags` 均为可选——文件路径本身已提供足够上下文，不要求 frontmatter 以保持向后兼容。

```yaml
---
description: React component conventions for the frontend package
tags: [react, components, frontend]
---
```

**何时使用 frontmatter：**

- 目录位置本身不足以描述文件用途（如根目录下多个 AGENTS.md 共享同一路径上下文）
- monorepo 含 5+ 个 AGENTS.md 文件，代理需要索引能力
- 文件指导范围足够特化，值得显式标注触发条件

**无需使用 frontmatter：**

- 仅含一个 AGENTS.md 的小项目——路径本身已足够
- 内容从文件名即可推断（如 `scripts/AGENTS.md` 显然与脚本相关）

> 此提案尚在草案阶段，非所有工具均已实现 frontmatter 感知。写入 frontmatter 不影响向后兼容——不识别的工具会忽略它。

---

## CLAUDE.md 专属指导

CLAUDE.md 是 Claude Code 原生读取的项目级配置文件。核心策略是 symlink 到 AGENTS.md 以维护单一真理源；当需要使用 `@import`、Commands 等 Claude Code 独有特性时，可维护独立文件。

详细策略、独有特性、Commands 目录说明见 [`references/claude-md.md`](references/claude-md.md)。

---

## .cursor/rules 格式

`.cursor/rules/` 目录使用 `.mdc` 文件格式，通过 `globs` 字段按文件匹配注入规则（`alwaysApply: true` 则常驻）。与 AGENTS.md 的职责划分：AGENTS.md 管全局约定，`.mdc` 管文件级规则。

详细格式、字段说明、职责对比、选用指南见 [`references/cursor-rules.md`](references/cursor-rules.md)。

---

## 机制层（Hooks / Subagents / Rules）

AGENTS.md / CLAUDE.md / `.cursor/rules` 都是**指令层**——依赖模型遵从，可被绕过。多种工具提供**机制层**用于确定性强制或隔离执行——这是通用原则 “An instruction asks, a mechanism requires”（见前置 Skill A.2）的落地。

> **工具归属**：以下机制（hooks / subagents / output styles / plugins）**多为 Claude Code 专属**；
> 其它代理（OpenCode / Cursor / Gemini CLI / Copilot）的等价或尚无等价见 [references/mechanism-layer.md](references/mechanism-layer.md) 的跨工具支持矩阵。
> 写跨工具配置时，优先用跨工具标准（AGENTS.md / SKILL.md），把 Claude Code 专属机制作“可选增强”。

| 层 | 机制 | 强制度 | 成本 |
|---|---|---|---|---|
| 指令层 | AGENTS.md / CLAUDE.md / rules | 依赖模型遵从 | 高（常驻）|
| 机制层 | Hooks / Permissions | 确定性（exit 2 阻断 / allow/deny/ask）| 低（配置在上下文外）|
| 隔离层 | Subagents / task agents | 隔离上下文 | 低（仅摘要回主会话）|

**核心判据**——“Never” 类规则该写在哪：

- 偶尔被违反也无大碍 → 指令层（AGENTS.md）
- **绝对不能被违反**（提交密钥、force push、删生产数据）→ **不要只写指令**；用 `PreToolUse` hook 阻断（exit 2），即使在 `bypassPermissions` 模式下也生效

八种指令方法完整决策表（加载时机 / 压缩行为 / 成本 / 适用）、hooks 五类型与生命周期、subagent vs skill 决策、`.claude/rules` 的 `paths:`、output styles、plugins、dynamic workflows、**OpenCode 权限模式（`permission.skill`）** 见 [references/mechanism-layer.md](references/mechanism-layer.md)。

---

## 写作原则

AGENTS.md 专属写作原则（通用规则见前置 Skill）：

- **Toolchain First**——确定性约束（代码风格、类型、构建、测试）归属工具链配置，AGENTS.md 只承载建议性指令（架构判断、工作流偏好）。

  ```
  # 好——指向工具，不重复规则
  Lint: `pnpm lint`（Biome——见 biome.json）
  # 坏——代替工具写规则
  不要用 var，始终用 const/let，import 顺序按标准库/三方/内部排列...
  ```

- **三层边界 Always / Ask / Never**——比简单禁令清单更有效，但维护中极易**膨胀**，应作为快速索引而非规则正文：
  - **Always Do**：每次自动执行（如提交前运行 `pnpm test`）
  - **Ask First**：重大变更先确认（如改数据库 schema）
  - **Never Do**：绝对禁止（如提交密钥、push main）——配肯定替代（见前置 Skill“肯定指令优先”）
  - **分小节放置**：用 `### Always` / `### Ask` / `### Never` 独立小节分类承载，代理跳读时可快速定位。混排在大表或单一列表中会削弱分类索引价值。
  - **保持索引精简**——每条一行、高信号：工具链能强制的（hook / CI / linter）不入此列，指向其配置（见 `Toolchain First`）；长解释下沉到引用文件。“只增不减”是膨胀主因，定期移除代理已能遵循的条目（见[维护流程](#维护流程)）。
- **反自动化生成**——LLM 自动生成的 AGENTS.md 一致降低成功率、推高推理成本（完整数据与机制见[L3]）。`/init` 等结果只当“内容清单”，手工重写。
- **关键文件路径显式标注**——入口点、基类、配置文件应显式标注路径。
- **@import 引用**——部分工具（如 Claude Code）支持 `@路径/文件名.md` 内联引用外部文件，根文件保持精简，知识按需加载。非 v1.1 标准特性（进展见 [cross-tool-compat.md](references/cross-tool-compat.md#标准化进展)），使用前确认工具兼容性。
- **重点标注非常规**——主流实践、常见配置等显而易见的内容一笔带过；非常规、反直觉、项目特有的内容重点提及。

---

## 决策表

当项目中存在两种或多种合理做法时，决策表能强制让代理在选择前做出决定，而不靠猜测。来自 [R5] 的最强模式之一——将决策表加入 AGENTS.md 后 `best_practices` 提升 25%。

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

**快速模板**：

```markdown
## [决策主题]选型

| 判断条件 | → [选项 A] | → [选项 B] |
|----------|:---------:|:---------:|
| [条件 1：如"需要服务器状态同步"] | ✅ 推荐 | |
| [条件 2] | | ✅ 推荐 |
| [条件 3：两者均可] | ✅ | ✅ |
```

每列放一个选项，每行一个判断条件，`✅` 标记该条件下推荐哪个选项。
避免空行/空列——表不完整时代理会靠猜测。

---

## 维护流程

### 增量迭代法

1. **起步**：仅覆盖最常出错的命令和边界，从简开始
2. **观察**：用真实任务记录代理反复出错的地方
3. **补充**：将反复出现的问题写入
4. **精简**：代理已能遵循的规则可移除
5. **重复**

**大小参考**：无最低行数要求；建议 100–150 行，不超过 200 行（Anthropic 官方建议 <200 行；[R1] 对 100 个热门仓库分析发现 42% 的文件 >200 行并出现 Context Bloat）。

**实证张力**——效果由内容质量与精简度共同决定，而非“有无”本身：手写精简有益（[R2] 降运行时间与 token、不损质量）；冗余与自动生成有害（[R3] 推高推理成本、一致降成功率）；而文件大小/位置/架构等结构变量对单条指令的即时遵从影响有限（[R4]，与成本证据测的维度不同、不冲突）。结论：精简、聚焦非显而易见内容是收益来源；冗余与自动生成是成本来源——完整数据见[L3]。

### 重构

提取/迁移目标按优先级：**先同文件相关章节**（就近归并，免一次加载跳转），**再子目录 / 子文件**（同文件无合适归处、或内容重量级时）。

- **审计**：遍历现有指令，标记可由工具链强制或已过时的内容
- **归并**：将错置内容移入当前文件最相关的非索引章节（优先于下沉）
- **拆分**：同文件无可归处时，按层级将臃肿内容分配到子目录
- **迁移**：将确定性规则移出 AGENTS.md，指向工具配置
- **验证**：改动前后对比，确认代理推理开销未增加

### 维护规则

AGENTS.md 专属：

- **写前检查**：遍历锁文件、CI 和现有代码模式再落笔。
- **定期审计**：周期性审查后将可被工具链强制的规则迁移出去（指向工具配置）。

---

## Monorepo 多 AGENTS.md 最佳实践

大型 monorepo 中常出现多个 AGENTS.md 文件（如 OpenAI 仓库已有数十个）。正确管理多文件的索引与发现对维持代理效率至关重要。

### 索引策略

- **根 AGENTS.md 做目录索引**：在根文件顶部列出所有子 AGENTS.md 的 path + 一句话概述，帮助代理预览可用的上下文范围。
- **子文件配 frontmatter**（可选）：为每个子 AGENTS.md 添加 YAML `description` 和 `tags`，代理可建立轻量索引而非全量加载（见上方[渐进式披露](#渐进式披露agentsmd-可选-frontmatter)）。

```yaml
---
description: API service build and test conventions for the payments team
tags: [payments, api, rust, ci]
---
```

### 增量审计流程

多 AGENTS.md 的维护分三步：

1. **索引审计**（每季度）——遍历所有 AGENTS.md 文件，检查是否存在：
   - 过时指令（参考的 API 已废弃、路径已迁移）
   - 与根文件或兄弟文件重复的规则
   - 超过 200 行且未配 frontmatter 的大文件
2. **精简回合**——删除重复规则、将长文件拆分子目录、补充缺失的 frontmatter
3. **交叉验证**——随机选 2–3 个目录测试代理是否加载了正确的 AGENTS.md 指导

### 文件命名约定

- 根文件：`AGENTS.md`
- 子目录文件：`<目录名>/AGENTS.md`（不额外命名，路径即标识）
- 避免同一目录出现多个 `AGENTS.*` 变体（如 `AGENT.md` + `AGENTS.md`），会造成优先级歧义

### 参考

Monorepo 多 AGENTS.md 的加载规则（管辖范围、累积、优先级）见上方[层级与作用域](#层级与作用域)。OpenAI 数十个 AGENTS.md 的实战案例见 [agents.md 官网](https://agents.md)。

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

各内容类型的分类（推荐 / 可选 / 不放）、详细维度评分与根 / 子目录拆分见[L1]（附录：详细目录）。

---

## 审计与质量

### 常见错误

| 错误 | 详见 |
|---|---|---|
| 含 README 内容 | 定位 |
| 重复工具链已强制内容（即 Lint Leakage）| 写作原则 Toolchain First / [配置异味检测](#配置异味检测) |
| 自动生成不审校（即 Init Fossilization）| 写作原则 反自动化生成 / [配置异味检测](#配置异味检测) |
| 不同工具各维护一份 | 维护唯一 AGENTS.md，symlink 到各工具入口文件（CLAUDE.md / GEMINI.md 等）|
| 否定指令 | 前置 Skill（肯定指令优先） |
| 边界规则混排在大表或单一列表 | 写作原则：三层边界 / 分小节放置 |
| Always/Ask/Never 小节越堆越长（边界膨胀）| 写作原则：三层边界 / 保持索引精简；工具可强制的指向工具，定期精简（见[维护流程](#维护流程)）|
| CLAUDE.md 与 AGENTS.md 不相关而用 symlink | CLAUDE.md 专属指导 |
| 把个人偏好写在项目级配置 | writing-agent-docs → 作用域 |
| .cursor/rules 无 glob 范围 | .cursor/rules 格式节 |

### 配置异味检测

[R1] 分析了 100 个热门仓库的 AGENTS.md，识别出六种常见配置异味。编写和审计时应检查：

| 异味 | 说明 | 检查方法 |
|------|------|---------|
| **Lint Leakage**（最普遍，62%） | 重复 linter/formatter 已强制的规则（命名风格、缩进、import 排序） | 被 Biome/ESLint/Ruff 等工具能自动修复的 → 删除 |
| **Context Bloat**（42%） | 文件 >200 行，堆砌规则和细节 | 先归入同文件相关章节；无合适归处再拆到子目录 AGENTS.md 或技能文件 |
| **Skill Leakage**（35%） | 把仅特定场景需要的指令放入常驻 AGENTS.md（如测试指南、脚手架流程） | 迁移到技能或子文件按需加载 |
| **Conflicting Instructions**（28%） | 指令互相矛盾，如两个不同路径指向同一职责 | 定期审计，冲突处显式注明优先级 |
| **Init Fossilization**（24%） | 由 `/init` 自动生成后从未人工审校修改 | 创建后至少人工审校一次，迭代更新 |
| **Blind Reference**（16%） | 引用外部文件时不说明用途和场景 | 每条引用配一句话：什么内容、何时读 |

**核心原则**：配置越精炼、越聚焦项目特有内容，代理表现越好。冗余指令每多一条，关键规则的注意力就少一分。

> 异味 Lint Leakage 和 Init Fossilization 在上方[常见错误表](#常见错误)中也有对应条目（重复工具链已强制内容、自动生成不审校），从实践角度互为补充。

---

## 安全考虑

AGENTS.md 注入代理上下文，因此也引入安全风险。编写时注意：

- **Blind Reference 可能泄露敏感文件**——引用 `../internal/credentials.md` 等路径时，代理可能会读取并加载其内容。确保引用文件本身可安全暴露给代理。
- **不暴露内部基础设施细节**——无需在 AGENTS.md 中记录内网 IP、内部域名、未公开 API 端点等。代理不需要这些信息来完成任务。
- **`/init` 生成后必审校**——自动生成的配置文件常包含过于宽泛的架构描述和内部路径，发布前应人工清理。
- **凭证永不写入**——API token、数据库密码、SSH 密钥等即使注释掉也不应出现在 AGENTS.md 或引用文件中。

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R1] | <https://arxiv.org/abs/2606.15828> | Context Bloat in AGENTS.md: An Empirical Study of 100 Repositories | 42% 仓库 AGENTS.md 超 200 行出现 Context Bloat |
| [R2] | <https://arxiv.org/abs/2601.20404> | Hand-crafted AGENTS.md Improves Efficiency Without Quality Loss | 手写 AGENTS.md 降低运行时间与 token 消耗 |
| [R3] | <https://arxiv.org/abs/2602.11988> | Redundant Instructions Increase Reasoning Costs in LLM Agents | 不必要指令推高推理成本 |
| [R4] | <https://arxiv.org/abs/2605.10039> | Positional Bias in LLM Instruction Following | 文件大小/位置对遵从无显著效应 |
| [R5] | <https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files> | How to Write Good agents.md Files | 决策表提升 best_practices 遵从率 25% |
| [R6] | <https://github.com/agentsmd/agents.md/issues/135> | AGENTS.md v1.1 Proposal | YAML frontmatter（description/tags）、渐进式披露、管辖/累积/优先级/继承四大语义 |

---

## 本地参考

| 编号 | 文件 | 内容 |
|------|------|------|
| [L1] | [references/content-decisions.md](references/content-decisions.md) | 附录：内容决策详细目录（维度评分 / 放入条件 / 根子目录拆分） |
| [L2] | [comparison-tools.md](../writing-agent-docs/references/comparison-tools.md) | AGENTS.md vs Skill vs MCP 对比（含 token 开销） |
| [L3] | [references/empirical-evidence.md](references/empirical-evidence.md) | AGENTS.md 实证数据（效率/成本/遵从，含自动生成危害与关键区分） |
| [L4] | [agent-persona.md](../writing-agent-docs/references/agent-persona.md) | Agent Persona 完整定义（项目级 + 个人级，跨技能共享） |
| [L5] | [references/cross-tool-compat.md](references/cross-tool-compat.md) | 跨工具概念对照 + AGENTS.md v1.1 标准化进展 |
| [L6] | [references/claude-md.md](references/claude-md.md) | CLAUDE.md 专属指导（Symlink 策略、独有特性、Commands 目录）|
| [L7] | [references/cursor-rules.md](references/cursor-rules.md) | .cursor/rules .mdc 格式（字段说明、与 AGENTS.md 职责划分）|
| [L8] | [references/mechanism-layer.md](references/mechanism-layer.md) | 机制层（Hooks / Subagents / Rules / Plugins + 七方法决策表）|
