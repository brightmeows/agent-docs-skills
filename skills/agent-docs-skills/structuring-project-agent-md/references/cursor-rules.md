# .cursor/rules 格式

> `structuring-project-agent-md` 参考文件。
> `.cursor/rules/` 目录使用 `.mdc` 文件格式，通过 glob 模式控制哪些文件触发哪些规则。
> **2026 更新：** Cursor v0.45+ 已正式弃用单文件 `.cursorrules`，改用 `.cursor/rules/*.mdc` 目录格式。
> 原有的 `.cursorrules` 仍向后兼容但优先级低于 `.mdc`，建议迁移。迁移步骤：
>
> 1. 创建 `.cursor/rules/` 目录
> 2. 将 `.cursorrules` 内容写入 `platform-base.mdc`（`alwaysApply: true`）
> 3. 删除 `.cursorrules`
> 4. 随时间将领域特定规则拆分为独立的 `.mdc` 文件（带 `globs`）

---

## 格式

```yaml
---
description: React 组件规则
globs: src/components/**/*.tsx
---
# React 组件

始终使用函数组件 + hooks，不用 class 组件。
导出用命名导出（named export），不用默认导出（default export）。
```

## 字段说明

| 字段 | 必需 | 说明 |
|------|------|------|
| `description` | 是 | 规则描述，Cursor 用它判断何时自动附加规则——`alwaysApply: false` 且无 globs 匹配时，Cursor 读此字段决定当前任务是否相关；`alwaysApply: false` 时有 globs 匹配时也用于辅助判断 |
| `globs` | 否 | 文件匹配模式，匹配到的文件才注入此规则 |
| `alwaysApply` | 否 | `true` 时忽略 globs，始终注入（类似 AGENTS.md 常驻）|

## 与 AGENTS.md 的职责划分

| 维度 | AGENTS.md | .cursor/rules/*.mdc |
|------|-----------|---------------------|
| **作用域** | 整个项目（常驻上下文）| 按文件匹配（按需注入）|
| **适用场景** | 全局约定、命令、边界 | 文件级编码规则、架构约束 |
| **加载方式** | 常驻 | glob 匹配时注入 / `alwaysApply` 常驻 |
| **优先级** | 子目录覆盖根 | glob 匹配广度影响注入时机 |

## 三层协同

| 文件 | 受众 | 管什么 | 加载方式 |
|------|------|--------|---------|
| `AGENTS.md` | **跨工具**（Cursor/Claude Code/Copilot 等均读）| 全局约定、命令、边界 | 常驻 |
| `.cursor/rules/*.mdc` | **Cursor 专属** | 文件级规则、领域特定约束 | glob 匹配注入 / `alwaysApply` 常驻 |
| `CLAUDE.md` | **Claude Code 专属** | 内存配置、MCP、斜杠命令、任务模板 | 常驻 |

三者内容不重复——`AGENTS.md` 管跨工具基线，`.mdc` 管 Cursor 专有行为（内联编辑响应、diff 结构等），`CLAUDE.md` 管 Claude Code 专有特性。

## 何时选用

- 规则适用于特定目录或文件模式 → `.mdc` 带 globs
- 规则适用于整个项目且跨工具共享 → AGENTS.md
- 规则需要始终强制 → `.mdc` 带 `alwaysApply: true`
- 规则只对 Cursor 内联编辑行为有意义 → `.mdc`（不在 AGENTS.md 中重复）
- 跨工具共享 → AGENTS.md（.mdc 主要为 Cursor/OpenCode 生态）

### 迁移引导

| 当前状态 | 目标状态 |
|---------|---------|
| 单一 `.cursorrules` | `.cursor/rules/platform-base.mdc`（`alwaysApply: true`）+ 按领域拆分 `.mdc` 文件 |
| AGENTS.md 含 Cursor 专有指令 | 移出到 `.mdc`，AGENTS.md 只保留跨工具内容 |
| CLAUDE.md 与 AGENTS.md 不相关 | 见 [cross-tool-compat.md](./cross-tool-compat.md#converge-to-single-truth)「从多份独立文件收敛到单一真理源」|

> OpenCode 兼容 `.cursor/rules/*.mdc` 格式，可作为项目级规则目录使用。

## 本地参考

（无本地引用）
