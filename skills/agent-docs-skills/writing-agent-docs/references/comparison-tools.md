# 配置机制职责对比

> `writing-agent-docs` 参考文件：配置机制的职责划分与 token 开销对比。本文件是"常驻 vs 按需"token 经济的唯一定义。
> 前三者（AGENTS.md / Skill / MCP）为**跨工具标准**；Hooks / Subagent 为 **Claude Code 专属**机制（详见 `structuring-project-agent-md` 机制层参考）。

| 用途 | 工具 | 示例 |
|------|------|------|
| 项目约定、命令、边界 | AGENTS.md | "用 pnpm，命名导出" |
| 多步骤工作流 | Skill | "部署上 staging → smoke test → 通知 Slack" |
| 确定性强制（阻断、自动 format）| Hooks（Claude Code 专属）| "PreToolUse exit 2 阻断 `rm -rf`、PostToolUse 跑 prettier" |
| 隔离执行侧任务 | Subagent（Claude Code 专属）| "深度搜索、日志分析、依赖审计，仅摘要回主会话" |
| 数据库查询、外部工具 | MCP Server | "@postgres 查询用户表" |

AGENTS.md 管**项目上下文**，Skill 管**任务知识**，Hooks 管**确定性强制**，Subagent 管**隔离执行**，MCP 管**外部工具**——五者互补。

**职责划分的权威定义**：[R1]明确"AGENTS.md focuses on **behavior** (rules, constraints, workflows); SKILL.md focuses on **capabilities**"——与本文件的划分一致。

**为何不把一切都塞进 AGENTS.md**：AGENTS.md 内容常驻上下文，而 Skill 按需加载。SkillsBench 实证（[R2]）显示，2–3 个聚焦技能比单一大文档（如把所有任务知识塞入 AGENTS.md）的 agent 通过率高出 +18.6 个百分点，而后者反而降低 -2.9 个百分点——任务知识放 Skill 能显著降低常驻开销并提升效果。

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|----------|
| [R1] | <https://github.com/agentsmd/agents.md/issues/135> | AGENTS.md v1.1 提案 | AGENTS.md 管行为与约束，SKILL.md 管能力——职责划分的权威定义 |
| [R2] | <https://arxiv.org/abs/2602.12670> | SkillsBench: A Benchmark for Agent Skill Evaluation | 2–3 个聚焦技能 +18.6pp vs 单一大文档 -2.9pp |

## 本地参考

| 编号 | 文件路径 | 用途 |
|------|----------|------|
| [L1] | [../SKILL.md](../SKILL.md) | writing-agent-docs 技能主文件，定义基础写作原则与约束 |
