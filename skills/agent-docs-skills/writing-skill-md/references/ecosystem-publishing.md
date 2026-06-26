# 技能生态与发布

> `writing-skill-md` 参考文件：技能生态概况、市场分布、发布流程与质量验证数据。

---

## 技能市场与注册中心

技能生态在 2026 年上半年经历了爆发式增长。根据 Skillselion 追踪数据（[R14]），
公开生态已达 **~66,000 个 agent skills、~7,800 个 MCP servers**，累计安装量 **112M**。
各市场索引规模因口径而异——SkillsMP 约 **190 万**、Skills.sh 约 60 万、ClawHub 约 1.3 万
（安全清查后余 3,200+）。质量参差——SkillsBench 评测 47,150 个公开技能平均 6.2/12（[R9]）。

主要市场分布：

| 平台 | 发布者 | 特点 |
|------|--------|------|
| **[Skills.sh](https://skills.sh)** | Vercel（2026-01） | CLI 安装（`npx skills install`）、Snyk 集成安全扫描、策展推荐 |
| **[ClawHub](https://clawhub.ai)** | 社区 | 自动索引 GitHub 公开 SKILL.md 文件、质量指标 |
| **[claude-plugins.dev/skills](https://claude-plugins.dev/skills)** | 社区 | 自动索引 Claude Code / Cursor / Codex 技能、开源 |
| **SkillsMP** | 第三方 | 企业级技能市场 |

## 如何发布技能

1. **遵循开放标准**：确保 SKILL.md 格式符合 [agentskills.io](https://agentskills.io/specification) 规范——所有市场均基于同一标准
2. **版本控制**：使用 Git tag 管理版本，发布时锁定到 release tag（`npx skills add <url>#v1.0.0`）
3. **GitHub 公开仓库**：将技能放在公开 GitHub 仓库的 `skills/` 目录下，市场将自动索引
4. **安全扫描**：发布前用 `mcp-scan` 扫描（`uvx mcp-scan@latest --skills`）
5. **description 优化**：按 CSO 原则编写 description，确保市场搜索能匹配到你的技能

## 技能生态验证

SkillsBench（[R9]）是首个 peer-reviewed 技能评估基准，基于 **87 个任务** × 11 个领域 × 7,308 条轨迹（v1.1 从 84 个扩至 87 个，采用原生 BenchFlow task.md 格式）。关键发现：

- **质量方差大**：47,150 个公开技能平均评分仅 6.2/12，仅 top-quartile（≥9 分）才有实质提升
- **精选技能有效**：精选技能提升通过率平均 16.2 个百分点（医疗领域 +51.9）
- **聚焦胜于臃肿**：2–3 个聚焦技能优于单一大文档（+18.6 vs -2.9）

可作为技能质量参考。

## 跨工具兼容性

SKILL.md 开放标准已被 **~40 工具** 原生支持，包括 Claude Code、OpenCode、Codex CLI、Cursor、
Gemini CLI、GitHub Copilot、Microsoft Agent Framework、JetBrains Junie、Goose、Amp、
Kiro、Roo Code、Factory、Databricks Genie Code、Snowflake Cortex Code、Spring AI 等
（完整列表见 [agentskills.io 展示页](https://agentskills.io/clients)）。写一次技能，跨平台可用。
OpenCode、Cursor 等额外支持 Claude Code 扩展字段的子集。

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R9] | <https://arxiv.org/abs/2602.12670> | SkillsBench: A Benchmark for Agent Skill Evaluation | 87 任务（v1.1）× 11 领域 × 7,308 轨迹；47,150 公开技能平均评分 6.2/12；精选技能提升通过率 +16.2pp |
| [R14] | <https://skillselion.com/state-of-ai-agent-skills-2026> | The State of AI Agent Skills 2026 | Skillselion 生态追踪：~66K skills、~7.8K MCP、112M 总安装量 |
| [R15] | <https://agentman.ai/blog/agent-skills-ecosystem-report-2026> | The Agent Skills Ecosystem in 2026 | 2026-06 生态系统报告：~40 兼容产品、190 万+ 公开技能、SkillsBench 6.2/12、安全审计 22,511 技能含 140,963 问题 |
