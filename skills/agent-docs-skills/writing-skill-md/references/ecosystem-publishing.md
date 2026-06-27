# 技能生态与发布

> `writing-skill-md` 参考文件：技能生态概况、市场分布、发布流程与质量验证数据。

---

## 技能市场与注册中心

技能生态已具规模——SKILL.md 是分发代理知识的标准载体，而非小众实验。
质量参差——精选技能有效，平均水平偏低（[R9]；精确数据见参考表 [R14]、[R15]）。

主要市场类型（代表平台见 [R15]）：

| 类型 | 特点 |
|------|------|
| **CLI 安装型**（如 Vercel Skills.sh） | `npx skills install`、Snyk 安全扫描、策展推荐 |
| **社区索引型**（如 ClawHub） | 自动索引 GitHub 公开 SKILL.md，质量指标 |
| **插件集合型**（如 claude-plugins.dev） | 按工具分类聚合技能，开源社区维护 |
| **全网抓取型**（如 SkillsMP） | 大规模 GitHub 扫描，覆盖面广、但无审查 |

## 如何发布技能

1. **遵循开放标准**：确保 SKILL.md 格式符合 [agentskills.io](https://agentskills.io/specification) 规范——所有市场均基于同一标准
2. **版本控制**：使用 Git tag 管理版本，发布时锁定到 release tag（`npx skills add <url>#v1.0.0`）
3. **GitHub 公开仓库**：将技能放在公开 GitHub 仓库的 `skills/` 目录下，市场将自动索引
4. **安全扫描**：发布前用 `mcp-scan` 扫描（`uvx mcp-scan@latest --skills`）
5. **description 优化**：按 CSO 原则编写 description，确保市场搜索能匹配到你的技能

## 技能生态验证

SkillsBench（[R9]）是首个 peer-reviewed 技能评估基准，基于 87 个任务 × 11 个领域 × 7,308 条轨迹。关键发现：

- **质量方差大**：数万公开技能平均评分偏低（满分 12），仅 top-quartile 才有实质提升
- **精选技能有效**：精选技能显著提升通过率（医疗领域提升尤为突出）
- **聚焦胜于臃肿**：少量聚焦技能优于单一大文档

可作为技能质量参考。

## 跨工具兼容性

SKILL.md 是开放标准——跨工具兼容是其设计目标，写一次即可跨平台使用。
完整兼容列表见 [agentskills.io 展示页](https://agentskills.io/clients)。
部分工具额外支持 Claude Code 扩展字段的子集。

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R9] | <https://arxiv.org/abs/2602.12670> | SkillsBench: A Benchmark for Agent Skill Evaluation | 87 任务（v1.1）× 11 领域 × 7,308 轨迹；47,150 公开技能平均评分 6.2/12；精选技能提升通过率 +16.2pp |
| [R14] | <https://skillselion.com/state-of-ai-agent-skills-2026> | The State of AI Agent Skills 2026 | Skillselion 生态追踪：84K+ 工具（65K skills、7.8K MCP、8.3K 市场）、112M 总安装量 |
| [R15] | <https://agentman.ai/blog/agent-skills-ecosystem-report-2026> | The Agent Skills Ecosystem in 2026 | 2026-06 生态系统报告：~40 兼容产品、190 万+ 公开技能、SkillsBench 6.2/12、安全审计 22,511 技能含 140,963 问题 |
