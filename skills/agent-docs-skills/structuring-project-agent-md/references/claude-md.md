# CLAUDE.md 专属指导

> `structuring-project-agent-md` 参考文件。
> CLAUDE.md 是 Claude Code 原生读取的项目级配置文件。本文件覆盖它与 AGENTS.md 的关系及独有特性。

---

## 与 AGENTS.md 的关系

| 策略 | 适用场景 | 做法 |
|------|---------|------|
| **Symlink** | AGENTS.md 内容完全适用于 Claude Code | `ln -s AGENTS.md CLAUDE.md`，维护单一真理源 |
| **独立文件** | Claude Code 需要项目级特有行为（Commands、@import） | 各自维护，定期同步重叠内容 |
| **.claude/ 目录** | 需要拆分多文件指令 | 项目根建 `.claude/` 目录，放入 `.md` 文件 |

## Symlink 优先

默认推荐 symlink 策略（`CLAUDE.md → AGENTS.md`），理由：

- 单一真理源，避免两份文件漂移
- 减少代理的配置复杂度
- AGENTS.md 是跨工具标准，其他工具也能读

## 何时需要独立 CLAUDE.md

- 需要使用 Claude Code 独有特性（见下方）
- AGENTS.md 内容被其他工具读取时不希望包含 Claude Code 特定指令
- 项目使用 `.claude/` 目录组织多文件

## CLAUDE.md 独有特性

| 特性 | 说明 | 示例 |
|------|------|------|
| **CLAUDE_GLOBAL.md** | 个人级全局指令，位于 `~/.claude/` | 全局语言/框架偏好 |
| **@import** | 内联引用外部 `.md` 文件 | `@ docs/architecture.md` |
| **Commands 目录** | 自定义斜杠命令 | `.claude/commands/deploy.sh` |
| **.claude/ 目录** | 多指令文件拆分 | `.claude/project.md` + `.claude/stack.md` |

> 注意：`@import` 非 AGENTS.md v1.1 标准特性，使用前确认工具兼容性。

## Commands 目录

Claude Code 支持在 `.claude/commands/` 下放可执行脚本作为斜杠命令：

```
.claude/commands/
├── deploy.sh    # /deploy
├── test.sh      # /test
└── lint.sh      # /lint
```

每个脚本应有清晰的文件头说明用途。
