# 技能组合模式（Skill Composition）

> `writing-skill-md` 参考文件：技能与技能之间的组合编排模式。
> 本文覆盖 4 种组合模式（顺序/并行/复合/条件），以及它与 Skill Bundle（清单级批量加载）的区别。
> 完整研究背景见 [R14]。

---

## 何时需要组合模式

当项目中存在多个技能，且它们之间存在以下关系时，需要考虑组合编排：

| 信号 | 说明 |
|------|------|
| **技能有固定调用顺序** | 如代码审查→部署→通知，顺序固定 |
| **需要并行执行** | 如同时搜索多个数据源后汇总 |
| **技能可嵌套复用** | 如"发送通知"被"部署"和"监控"两个技能都引用 |
| **路由逻辑复杂** | 不同条件走不同技能路径 |

**关键区分**：Skill Bundle（清单式批量加载）解决的是"同时加载哪些技能"；Skill Composition 解决的是"技能间如何配合执行"。两者互补——Bundle 决定加载集，Composition 决定执行流。

---

## 四种组合模式

### 1. 顺序管道（Sequential Pipeline）

技能链式执行，前一个的输出作为后一个的输入。适用于流程固定的多步骤工作流。

```
规划 skill → 执行 skill → 审查 skill → 发布 skill
```

**在 SKILL.md 中表达**：

```markdown
## 工作流

1. 运行 `plan` skill 生成实施计划
2. 计划确认后，运行 `execute` skill 按计划执行
3. 执行完成后，运行 `review` skill 审查变更
4. 审查通过后，运行 `deploy` skill 部署

**错误处理**：任意步骤失败 → 停止并报告，不继续下游
```

**最佳实践**：

- 每个步骤定义明确的完成标准和错误处理
- 步骤间通过文件/环境变量传参，而非依赖上下文残留
- 顺序固定时用编号列表，非固定时用决策表（见 `structuring-project-agent-md` 决策表模式）

### 2. 并行扇出（Parallel Fan-out）

同时执行多个独立技能，然后汇总结果。适用于多源研究、多角度分析、并行审查。

```
             ┌─ 搜索 skill A ─┐
规划 skill ──┼─ 搜索 skill B ─┼── 综合 skill
             └─ 搜索 skill C ─┘
```

**在 SKILL.md 中表达**：

```markdown
## 并行分析

1. 同时启动以下任务（使用 subagent / task agent 隔离执行）：
   - 安全审查 → `security-review` skill
   - 性能评估 → `performance-review` skill
   - 代码质量 → `code-quality` skill
2. 等待所有任务完成
3. 运行 `synthesize` skill 汇总三份报告

**注意**：并行任务应隔离执行（Claude Code subagent / OpenCode task agent），
避免上下文污染。主 skill 仅协调和汇总。
```

**最佳实践**：

- 并行任务必须独立（互不依赖输入输出）
- 使用隔离执行机制（subagent / task agent）防止上下文污染
- 设置超时兜底（如"超过 5 分钟未返回视为超时"）
- 综合步骤需要定义冲突处理策略（结果矛盾时以谁为准）

### 3. 复合技能（Composite / Fractal Composition）

技能嵌套调用其他技能，形成层次化架构。复杂技能分解为可复用的子技能。

```
部署 skill
├── 构建 skill（可复用：被部署 + CI 共用）
├── 测试 skill（可复用：被部署 + 本地开发共用）
└── 发布 skill
    ├── 更新版本号 skill（可复用）
    └── 通知 skill（可复用：被发布 + 监控共用）
```

**在父 SKILL.md 中表达**：

```markdown
## 部署流程

1. 调用 `build` skill 构建产物
2. 调用 `test` skill 运行集成测试
3. 测试通过后：
   a. 调用 `bump-version` skill 更新版本号
   b. 调用 `publish` skill 发布到注册表
4. 调用 `notify` skill 通知团队

**子技能职责**：每个子技能只做一件事，通过 exit code / 输出文件传递结果。
```

**最佳实践**：

- **粒度原则**：子技能应该可独立测试、可独立复用。如果某个子技能只被一个父技能调用，考虑内联而非拆分
- **接口契约**：子技能的输入（前置条件）和输出（完成信号）需显式声明
- **版本同步**：修改子技能时检查所有调用它的父技能是否需要适配
- **避免过深嵌套**：建议不超过 2 层（父→子→孙），超过则考虑扁平化

### 4. 条件分发（Conditional Dispatch）

根据运行时条件选择不同的技能路径。适用于路由逻辑复杂、多分枝的场景。

```markdown
## 路由逻辑

- 如果 incoming 包含 "health-check" → 调用 `health-check` skill
- 如果 incoming 是 "Heartbeat" → 直接确认，不调用其他 skill
- 否则：
  - 分类意图
  - 根据分类路由到对应 skill（`bug-fix`、`feature`、`refactor`）
```

**在 SKILL.md 中表达**：

```markdown
## 问题分类与路由

| 条件 | → 路由到 |
|------|---------|
| 报告为 bug | `bug-triage` skill |
| 新功能请求 | `feature-plan` skill |
| 性能问题 | `perf-diagnosis` skill |
| 安全问题 | `security-review` skill |
| 以上皆非 | 询问用户澄清 |

路由在 SKILL.md 中显式声明，代理可查表决策，无需猜测。
```

**最佳实践**：

- 条件用决策表而非文字描述（结构化优于叙述）
- 始终包含 fallback 分支（"以上皆非"）
- 条件互斥时用编号 if/else 链；非互斥时用决策表

---

## 组合模式的选型指南

| 场景 | 推荐模式 | 原因 |
|------|---------|------|
| 流程步骤固定（如部署）| 顺序管道 | 步骤间有明确依赖关系 |
| 多源研究、多角度分析 | 并行扇出 | 各任务独立，需要综合 |
| 有可复用的通用步骤 | 复合 | 避免重复，单一真理源 |
| 输入类型决定处理方式 | 条件分发 | 不同输入需不同处理流程 |
| 以上混合 | 嵌套组合 | 如：条件分发内各分支用顺序管道 |

---

## 跨工具考虑

组合模式的实现能力因工具而异：

| 执行方式 | Claude Code | OpenCode | Cursor |
|---------|-------------|---------|--------|
| **顺序执行**（串行调用）| ✅ SKILL.md 直接写顺序步骤 | ✅ 同左 | ✅ 同左 |
| **隔离并行** | ✅ subagent | ✅ task agent | ❌ 需 composer |
| **复合嵌套** | ✅ skill 内可引用其他 skill | ✅ 同左 | 有限 |
| **动态编排** | ✅ Dynamic Workflow | ❌ | ❌ |

**写跨工具组合时**：

- 顺序管道 + 条件分发是跨工具安全的（纯指令，不依赖专有机制）
- 并行扇出 + 隔离执行标记为工具特定（标注支持范围）
- 复杂编排可考虑 Claude Code Dynamic Workflow 做动态 harness

---

## 反模式

| 反模式 | 问题 | 正确做法 |
|--------|------|---------|
| **巨型技能** | 一个 SKILL.md 塞入所有子流程，>500 行 | 拆分为子技能，父技能仅做编排 |
| **过早拆分** | 一个 30 行技能硬拆成 5 个 | 技能内联，真需要复用时再拆分 |
| **循环引用** | skill A 调 skill B，skill B 又调 skill A | 消除循环，提取公共子技能 |
| **隐式依赖** | 子技能假设父技能已做了某操作 | 显式声明前置条件 |
| **忽略退出路径** | 组合未定义失败时的降级策略 | 每个步骤配错误处理 |

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R14] | <https://zylos.ai/research/2026-05-12-agent-skill-composition-modular-capability-architecture> | Agent Skill Composition: The Architecture of Modular AI Capabilities | 四种组合模式（顺序/并行/复合/条件）+ 生产部署最佳实践 + 跨工具兼容性分析 |

---

## 本地参考

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [ecosystem-publishing.md](./ecosystem-publishing.md) | 技能生态概况与发布流程 |
| [L2] | [anthropic-best-practices.md](./anthropic-best-practices.md) | 官方最佳实践补充——自由度、可执行脚本、MCP 引用 |
