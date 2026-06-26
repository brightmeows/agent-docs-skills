# .cursor/rules 格式

> `structuring-project-agent-md` 参考文件。
> `.cursor/rules/` 目录使用 `.mdc` 文件格式，通过 glob 模式控制哪些文件触发哪些规则。

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
| `description` | 是 | 规则描述，代理通过它决定是否加载 |
| `globs` | 否 | 文件匹配模式，匹配到的文件才注入此规则 |
| `alwaysApply` | 否 | `true` 时忽略 globs，始终注入（类似 AGENTS.md 常驻）|

## 与 AGENTS.md 的职责划分

| 维度 | AGENTS.md | .cursor/rules/*.mdc |
|------|-----------|---------------------|
| **作用域** | 整个项目（常驻上下文）| 按文件匹配（按需注入）|
| **适用场景** | 全局约定、命令、边界 | 文件级编码规则、架构约束 |
| **加载方式** | 常驻 | glob 匹配时注入 / `alwaysApply` 常驻 |
| **优先级** | 子目录覆盖根 | glob 匹配广度影响注入时机 |

## 何时选用

- 规则适用于特定目录或文件模式 → `.mdc` 带 globs
- 规则适用于整个项目 → AGENTS.md
- 规则需要始终强制 → `.mdc` 带 `alwaysApply: true`
- 跨工具共享 → AGENTS.md（.mdc 主要为 Cursor/OpenCode 生态）

> OpenCode 兼容 `.cursor/rules/*.mdc` 格式，可作为项目级规则目录使用。

## 本地参考

（无本地引用）
