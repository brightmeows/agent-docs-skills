# AGENTS.md vs Skill vs MCP

> structuring-project-agent-md 参考文件：AGENTS.md / Skill / MCP 三者的职责划分与 token 开销对比。

| 用途 | 工具 | 示例 |
|---|---|---|
| 项目约定、命令、边界 | AGENTS.md | “用 pnpm，命名导出” |
| 多步骤工作流 | Skill | “部署上 staging → smoke test → 通知 Slack” |
| 数据库查询、外部工具 | MCP Server | “@postgres 查询用户表” |

AGENTS.md 管**项目上下文**，Skill 管**任务知识**，MCP 管**外部工具**——三者互补。

**为何不把一切都塞进 AGENTS.md**：AGENTS.md 内容常驻上下文，而 Skill 按需加载。开发者内部实测显示，等效内容作为 AGENTS.md 常驻条目相对于作为 Skill 按需加载，每轮 token 开销约高 18 倍——任务知识放 Skill 能显著降低常驻开销。
