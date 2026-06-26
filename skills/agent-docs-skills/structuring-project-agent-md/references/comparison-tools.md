# AGENTS.md vs Skill vs MCP

> structuring-project-agent-md 参考文件：AGENTS.md / Skill / MCP 三者的职责划分与 token 开销对比。

| 用途 | 工具 | 示例 |
|---|---|---|
| 项目约定、命令、边界 | AGENTS.md | "用 pnpm，命名导出" |
| 多步骤工作流 | Skill | "部署上 staging → smoke test → 通知 Slack" |
| 确定性强制（阻断、自动 format）| Hooks（Claude Code 专属）| "PreToolUse exit 2 阻断 `rm -rf`、PostToolUse 跑 prettier" |
| 隔离执行侧任务 | Subagent（Claude Code 专属）| "深度搜索、日志分析、依赖审计，仅摘要回主会话" |
| 数据库查询、外部工具 | MCP Server | "@postgres 查询用户表" |

AGENTS.md 管**项目上下文**，Skill 管**任务知识**，Hooks 管**确定性强制**，Subagent 管**隔离执行**，MCP 管**外部工具**——五者互补。

**职责划分的权威定义**：[AGENTS.md v1.1 提案](https://github.com/agentsmd/agents.md/issues/135)明确“AGENTS.md focuses on **behavior** (rules, constraints, workflows); SKILL.md focuses on **capabilities**”——与本文件的划分一致。

**为何不把一切都塞进 AGENTS.md**：AGENTS.md 内容常驻上下文，而 Skill 按需加载。开发者内部实测显示，等效内容作为 AGENTS.md 常驻条目相对于作为 Skill 按需加载，每轮 token 开销约高 18 倍——任务知识放 Skill 能显著降低常驻开销。
