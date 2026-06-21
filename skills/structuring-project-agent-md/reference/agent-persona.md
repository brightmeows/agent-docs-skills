# Agent Persona（角色定义）

> structuring-project-agent-md 参考文件：Agent Persona 的三种定义模式（specialist / Registry / 单角色）与个人级扩展。

## 1. 定义 specialist 角色

```
# 正确——定义角色
你是一个 Rust 后端开发者。对安全性有最高优先级。需要 unsafe 代码时先提方案。

# 错误——模糊描述
你是一个帮助编码的助手。
```

角色帮助代理在权衡时做正确决定（安全 > 性能？可读性 > 巧妙？）。

## 2. Registry 模式（多角色场景）

若项目使用多个 agent 角色，在 AGENTS.md 中**只注册名称和调用方式**，完整定义放在 skill 文件中——避免每次会话加载所有角色的完整定义：

```
## Personas
Invoke via skill: @Lead, @Dev, @Critic
Definitions: `.claude/skills/`
```

## 3. 单角色项目

单角色项目保持简单：

```
## Identity
Senior Systems Engineer — Go 1.22, gRPC, high-throughput concurrency.
Favor explicit error handling and composition over inheritance.
```

---

## 4. 个人级 Persona（在个人配置文件中定义）

个人级 Persona 表达**你希望代理以什么角色为你工作**，独立于任何项目。

### 位置

| 工具 | 文件 |
|------|------|
| OpenCode | `~/.config/opencode/opencode.json` 中的 `agent` 定义或 `.opencode/agent/*.md` |
| Claude Code | `~/.claude/CLAUDE_GLOBAL.md` |
| Cursor | `~/.cursor/rules/`（全局规则文件）|

### 个人级 Persona 示例

```
# ~/.claude/CLAUDE_GLOBAL.md

你是一个有 15 年经验的全栈工程师。
- 偏好 TypeScript、Rust、Go
- 测试优先，提交前必跑测试
- 代码简洁 > 巧妙
- 文档和代码同等重要
```

### 个人级 Persona 写作原则

- **持续于项目**——不写项目特有的引用
- **表达偏好，非规则**——语言偏好、工作流习惯、编码哲学
- **保持简短**——5–10 行，太长会稀释项目级配置的注意力

---

## 5. 层级叠加

个人级 Persona → 项目级 Persona → 技能级 Persona（如有），按此顺序**累积**：

```
个人级（~/.claude/CLAUDE_GLOBAL.md）：
  "你是一个全栈工程师，偏好 TypeScript"

项目级（./AGENTS.md）：
  "本项目使用 Go 1.24 + Connect RPC"

→ 代理表现为：一个有 TypeScript 偏好的全栈工程师，
  在当前项目中用 Go + Connect RPC 工作。
```

### 冲突处理

| 冲突类型 | 优先级 |
|---------|--------|
| 个人级 vs 项目级 | 项目级胜出（就近优先）|
| 项目级 vs 技能级 | 技能级胜出（技能 body 含 HARD GATE）|
| 同级冲突 | 后加载的覆盖先加载的 |

**例**：个人级说“测试优先”，项目级说“本项目无需测试”——项目级胜出。若技能中写了 HARD GATE “在修改前必须先写测试”，则技能级覆盖项目级。

### 设计时的建议

- **项目级 Persona**：仅定义与项目强相关的角色属性（技术栈、领域知识）
- **个人级 Persona**：定义你个人的编码哲学和语言偏好
- **让差异显式化**：如果个人级和项目级有冲突，确保冲突是刻意的、有理由的
