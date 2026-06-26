---
name: writing-agent-docs
description: 提供代理文档写作的通用基础原则与约束。在创建、编辑或审校 AGENTS.md、SKILL.md、.cursor/rules、系统提示词等面向代理的文本，或不确定文档该写什么、怎么写时使用。是所有代理文档写作 skill 的前置依赖。
license: Apache-2.0
---

# 编写代理文档——通用规则

## 定位

本技能是**领域无关的代理文档 / 提示词写作基础**，适用于：

- AGENTS.md / SKILL.md
- `.cursor/rules` / `.junie/guidelines.md`
- 系统提示词
- 任何 agent-facing 文本（指南文件等）

领域专属规则见后续 skill：

- AGENTS.md / CLAUDE.md / .cursor/rules（项目级配置）→ `structuring-project-agent-md`
- SKILL.md（技能文档）→ `writing-skill-md`
- 个人级配置（~/.claude/CLAUDE_GLOBAL.md 等）→ `structuring-personal-agent-md`

---

## 作用域

面向代理的文档分**项目级**（仓库内 AGENTS.md / CLAUDE.md / .cursor/rules / GEMINI.md 等）
和**个人级**（~/.claude/CLAUDE_GLOBAL.md / ~/.agents/AGENTS.md 等）。
各作用域的文件清单、写作目标、加载顺序见[L1]。

---

## 核心原则

- **上下文是公共资源**：文档与系统提示、对话历史、其他元数据抢同一个有限注意力预算
- **每 token 须自证价值**：目标是找到“最大化期望结果的最小高信号 token 集”

> 这些原则是 **context engineering**（结构化、维护、治理塑造 AI 行为的信息）的具体应用——框架见 [R1]。

质疑每条信息：

- 模型真的需要吗？
- 能假设它已知吗？
- 这段文字值得它的 token 成本吗？

---

## 通用规则

### A. 选材域 — 决定写什么

#### A.1 不写推测性规则——增量迭代

- 只有当代理反复犯同一错误才添加
- 不要为假设场景写规则
- 起步极简 → 用真实任务观察 → 补充反复出现的问题 → 精简已能遵循的 → 重复
- “best docs grow through iteration, not upfront planning.”
- 当需要补充内容时，优先记录**具体踩坑点**（环境特有、反直觉的事实）而非通用建议——gotchas 是迭代中最直接的改进（[R7]）

#### A.2 确定性约束优先——能用机制强制的，不写入文档

- 约束可被机制确定性校验或强制（linter、类型系统、schema、校验脚本、hook）→ 交给机制，不写入文档
- 文档易过时、易被绕过；机制是确定性兜底
- “An instruction asks, a mechanism requires.”
- 文档只承载需判断的内容（架构决策、工作流偏好）

```markdown
# 坏：把可校验的规则写进文档
字段名必须全小写、用下划线、不超过 32 字符……
# 好：指向校验机制
字段命名由 schemas/validate.py 强制；此处仅记例外
```

- 项目工具链 → `structuring-project-agent-md`
- 技能脚本 → `writing-skill-md`

#### A.3 不写易于获取的内容——代理能自行发现的，不写进文档

代理已有预训练知识，且能直接读取代码、类型定义、目录结构等。不要重复这些——浪费 token 且会与源码漂移。

```markdown
# 坏：记录标准语法
"Python 函数用 def 关键字定义……"
# 好：仅记录项目特有内容
"此项目用 `def` 但必须加类型注解；函数超过 30 行应拆分"
```

- 代理已预训练：标准语法、常见 API、设计模式
- 代理能读：类型定义、函数导出、目录结构、现有文档

#### A.4 优先整体约束而不是具体细节——给约束，让代理自己推理

与其穷举具体规则，不如给一条总体约束让代理结合代码推断。约束更抗漂移、更易维护。

```markdown
# 坏：穷举细节
变量 camelCase、类 PascalCase、常量 UPPER_CASE、文件 kebab-case……
# 好：给总体约束
依循项目已有的命名约定（见现有代码 + .editorconfig）
```

- 适用：命名约定、代码风格、错误处理模式、架构原则——代理可从示例推理时
- 不适用：代理无法从上下文推断的非显而易见规则（需显式写明）

### B. 雕琢域 — 决定怎么写

#### B.1 简洁优先

- 能用一句话说清绝不用一段
- **命令优先**：指令写成精确的可执行命令（`pytest -v --tb=short`），而非描述性文字（“运行测试”）——命令可自验证，描述需要代理推断（[R6]）
- **提供默认而非菜单**：多个方案时指定一个默认，备选一笔带过，避免代理逐一尝试（[R7]）
- 原则是每 token 须自证价值；大小目标因文档类型而异（见领域 skill）

```markdown
# 坏：冗余解释（约 150 tokens）
PDF 是一种常见文件格式，包含文本、图像……要从 PDF 提取文本，你需要用一个库……
# 好：简洁（约 50 tokens）
## 提取 PDF 文本
用 pdfplumber：
with pdfplumber.open("file.pdf") as pdf: ...
```

#### B.2 肯定指令优先，否定慎用

- 告诉 LLM“不要做 X”反而强化对其的 attention（Pink Elephant Problem）——提及禁用词本身就会 prime 模型产生它（[R2]）
- **默认改写为肯定**；硬性禁令须保留否定时，**必须配一个明确的肯定替代**

```markdown
# 坏：纯否定
不要删除文件。
# 好：禁令 + 肯定替代
不要删除文件。改移到 ./trash/。
```

- 指令越多，否定的劣势越大；指令少于 5–6 条时差距小
- **例外——可 grep 的约束**：当约束可被确定性检查（grep / lint 可验证）时，否定形式更精确——禁令可 grep 确认，等效的肯定表达无法被自动校验（[R3]）

#### B.3 示例驱动

- **正反示例优先于纯文字解释**——一个正反示例胜过三段文字描述
- 示例要完整可运行、来自真实场景、注释解释“为什么”
- **避免“WRONG:”反例单独出现**——它会将要避免的模式注入上下文；有正确示例时，无需反例
- **示例用真实值，禁用占位符**——代理会字面复制占位字符串（[R4]：“Placeholder values are landmines.”）；禁用 `"string"`、`"YOUR_VALUE"`、`"example-slug"`，使用项目真实数据

```markdown
# 好：正反对比，一图胜千言
// 正确：命名导出，const 优先
export const formatDate = (date: Date): string => { ... }
// 错误：默认导出，var 声明
export default function formatDate(date){ var result; ... }
```

#### B.4 命名与路径规范

- **正斜杠**（跨平台）：`references/guide.md`，不用 `references\guide.md`
- **描述性命名**：`form_validation_rules.md`，不用 `doc2.md`
- **行号引用禁用**：用类型名 / 函数名 / 模块名，不用“第 42 行”
- **命令带精确 flag**：`pytest -v` 而非“跑测试”

#### B.5 术语一致 + 避免过时信息

- 术语全程一致（始终“字段 / field”，不混用 field / box / element / control）
- 不写会过时的信息（版本号、日期、临时 API）；必要时用“旧模式 / Legacy”折叠区
- **过时指令比没有更糟**——像代码一样维护，架构变更时同步更新

### C. 架构域 — 决定怎么组织

#### C.1 验证反馈循环

- 运行验证器 → 修复 → 重复
- 所有路径与命令须真实存在
- 关键操作有验证 / 确认步骤

```markdown
1. 完成改动
2. 立即验证（运行测试 / lint / 校验脚本）
3. 失败则审查错误 → 修复 → 再验证
4. 仅在验证通过后继续
```

#### C.2 渐进式披露——拆分 + 按需加载

重内容下沉到子文件 / 子目录，入口保持精简，按需加载。具体机制（文件系统层级 / 三级加载 / 引用深度）见领域 skill。

#### C.3 结构与信息架构

- Markdown 分节
- **原子任务**：不合并可独立完成的步骤
- **bookend**：关键规则放文件顶部和任务区前（长上下文中部 attention 最弱）
- 注意：位置对单条指令的即时遵从影响有限（[R5]），会话长度才是关键——bookend 服务可读性而非保证即时遵从
- 步骤仅用于真正有顺序的工作流；并列任务用 bullet 不用编号（避免引入伪顺序）
- **按目标任务组织**：不按组件/端点/函数组织文档，而按代理要完成的任务组织。
  任务文档包含完整步骤序列，消除顺序歧义（[R4]：
  “Endpoint docs answer 'how do I call this?' Workflows answer 'how do I accomplish this?'”）

#### C.4 优先结构化而非逐条列举

同等信息量下，优先用结构化格式（表格、具名区块、key-value 对、代码块 + 标签头）而非纯逐条 bullet list。

- 代理解析结构化数据比解析无序列表更可靠——表格的行列关系、具名区块的标签是显式信号
  （[R4]：“Agents extract structured data from markdown tables easily.
  Bullet lists with mixed formatting are much harder to parse consistently.”）
- 逐条积累的规则列表是反模式——它把约束平铺成线性序列，丢失了类别、优先级和层次关系；有效指令应构成**相互作用的约束系统**而非规则清单（[R3]）
- 长 bullet list 应重组为带标头的节或表格

```markdown
# 坏：纯逐条列举（条件与行动隐含，代理需自行解析映射）
- 用户请求删除文件时，先确认再执行
- 用户请求修改配置时，先备份原文件
- 用户请求运行危险命令时，先解释风险

# 好：结构化表格（列标签确定义条件→行动映射）
| 用户请求 | 代理执行 |
|----------|----------|
| 删除文件 | 先确认，再执行 |
| 修改配置 | 先备份原文件 |
| 运行危险命令 | 先解释风险，再询问确认 |
```

#### C.5 工作流闭包——完成标准 + 阻塞升级

代理最常见的失败模式是未经验证就报告“完成”，以及在阻塞时采取破坏性变通方案。每条工作流应显式定义完成标准和异常路径。

- **完成标准**：定义具体的可验证条件，代理据此自检后再报告完成。“完成 = lint 通过 + 测试通过 + 已提交”而非“完成 = 改完代码”
- **阻塞升级**：说明代理在阻塞时应该做什么（和不该做什么）。“测试失败 3 次后停止并报告完整输出”“遇到冲突时停止并显示冲突文件”——禁止删除文件绕过错误

```markdown
# 坏：开放式的完成标准
确保代码质量后再提交

# 好：可验证的完成标准 + 升级路径
完成标准：
1. `ruff check .` 返回 0
2. `pytest -v` 全通过
3. 已提交，commit message 符合 conventional commits

阻塞时：
- 测试失败 3 次 → 停止并报告失败测试完整输出
- 依赖缺失 → 先查 requirements.txt，再问
- 绝不：删除文件解决错误、force push、跳过测试
```

---

## 自检清单（部署前）

**选材域：**

- [ ] 无推测性规则（只针对观察到的真实失败）；如补充内容，优先具体 gotchas
- [ ] 可机制强制的约束交给机制，未写入文档
- [ ] 无易于从代码/预训练获取的重复内容
- [ ] 给整体约束而非穷举细节

**雕琢域：**

- [ ] 每条信息通过“模型真的需要吗？”质疑
- [ ] 否定指令配了肯定替代（或确属硬性禁令）；可 grep 的约束优先否定形式
- [ ] 示例驱动——含正反对比且无占位符
- [ ] 术语全程一致
- [ ] 命令带精确 flag，路径用正斜杠
- [ ] 引用真实存在的路径与命令

**架构域：**

- [ ] 重内容下沉子文件，入口精简、按需加载
- [ ] 关键操作有验证 / 确认步骤
- [ ] 工作流有显式完成标准和阻塞升级路径
- [ ] 组织检查——无长串无序 rule list；结构化格式优先，按目标任务组织

> 工具链优先、Always/Ask/Never 边界、反自动生成、行数目标、一层引用深度等**领域专属规则**见 structuring-project-agent-md / writing-skill-md。

---

## 何时升级到领域 skill

通用规则之上：

- 写 **AGENTS.md / CLAUDE.md / .cursor/rules**（项目级配置）→ 加载 `structuring-project-agent-md`
- 写 **SKILL.md**（TDD、技能类型、CSO、三级渐进式披露、一层引用深度、<500 行目标）→ 加载 `writing-skill-md`
- 写 **~/.claude/CLAUDE_GLOBAL.md / ~/.agents/AGENTS.md**（个人级配置）→ 加载 `structuring-personal-agent-md`

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R1] | [anthropic.com](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Effective Context Engineering for AI Agents | 结构化上下文管理框架的官方指南 |
| [R2] | [arXiv:2601.08070](https://arxiv.org/abs/2601.08070) | Pink Elephant Punished: The Negative Instruction Priming Effect in LLMs | 否定指令会 prime 模型产生被禁止的行为 |
| [R3] | [agentpatterns.ai](https://agentpatterns.ai/training/foundations/prompt-engineering/) | Prompt Engineering for Agent Instructions and Systems | 指令应构成约束系统而非规则清单 |
| [R4] | [vercel.com](https://vercel.com/academy/agent-friendly-apis/agent-friendly-docs) | Agent-Friendly Docs | 代理解析结构化数据比无序列举更可靠 |
| [R5] | [arXiv:2605.10039](https://arxiv.org/abs/2605.10039) | Positional Bias in LLM Instruction Following | 指令位置对遵从影响有限，会话长度是关键 |
| [R6] | [blakecrosley.com](https://blakecrosley.com/blog/agents-md-patterns) | AGENTS.md Patterns: What Actually Changes Agent Behavior | 命令优先指令、完成标准定义、阻塞升级路径 |
| [R7] | [github.com/mgechev](https://github.com/mgechev/skills-best-practices) | Skills Best Practices | Gotchas 模式、提供默认而非菜单、Plan-Validate-Execute |

---

## 本地参考

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [scope-and-loader.md](./references/scope-and-loader.md) | 作用域与加载顺序的唯一定义（文件清单、流水线、优先级、两类作用域对比）|
| [L2] | [agent-persona.md](./references/agent-persona.md) | Agent Persona 定义（项目级 + 个人级，跨技能共享）|
