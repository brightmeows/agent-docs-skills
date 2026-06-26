# 生态全景——Agent Skills / MCP / Marketplaces

> `writing-agent-docs` 参考文件：技能生态中三类工具的职责划分、组合模式和选型指南。
> 数据来源：[Skillselion State of AI Agent Skills 2026][R1]。

---

## 生态构成

截至 2026 年中，公开追踪的 AI agent 工具生态由三大类构成：

| 类型 | 数量 | 累计安装 | 职责 |
|------|------|---------|------|
| **Agent Skills** | ~66,000 | 领先 | 指令与工作流（SKILL.md），教会 agent **做什么** |
| **MCP Servers** | ~7,800 | 快速增长 | 外部工具与数据连接，给 agent **用什么** |
| **Marketplaces** | ~8,300 | 生态基础 | 分发、发现与安全扫描 |

来源：[R1] Skillselion Catalog（skills.sh 注册表 + GitHub）。

---

## 三类工具的职责划分

### Agent Skills（指令层）

- **核心产物**：SKILL.md（YAML frontmatter + Markdown body）
- **加载机制**：三级渐进披露（元数据常驻 → body 按需 → 文件按需）
- **作用域**：项目级（仓库内 `skills/`）+ 个人级（`~/.claude/skills/` 等）
- **典型用途**：代码审查、部署流程、TDD、安全审计、语言最佳实践

### MCP Servers（工具层）

- **核心产物**：MCP 协议实现的 server（可本地或远程）
- **加载机制**：通过工具配置注册，agent 在需要时调用
- **作用域**：项目级（`opencode.json` 中定义）+ 个人级（全局 MCP 配置）
- **典型用途**：数据库查询、文件系统操作、API 调用、Web 搜索、图像生成

### Marketplaces（分发层）

- **核心产物**：技能/MCP 的可搜索目录
- **发现机制**：GitHub 公开仓库索引 + 注册表 + 策展推荐
- **安全扫描**：各市场提供不同程度的自动化扫描（Snyk / mcp-scan 等）
- **典型市场**：Skills.sh（Vercel）、ClawHub（社区）、SkillsMP、claude-plugins.dev

---

## Skill + MCP 组合模式

Skill 和 MCP 互补而非竞争。典型的组合模式：

| 模式 | Skill 的角色 | MCP 的角色 | 示例 |
|------|-------------|------------|------|
| **数据驱动工作流** | 定义流程步骤和验证标准 | 提供数据源和操作接口 | 部署 skill + 数据库 MCP |
| **智能审计** | 定义检查规则和报告格式 | 提供代码仓库和 CI 数据 | 安全审查 skill + GitHub MCP |
| **多源研究** | 定义研究方法和综合规则 | 提供搜索和文档获取 | 深度研究 skill + WebSearch MCP |
| **自动化运维** | 定义告警处理和故障恢复流程 | 提供监控和云服务接口 | 运维 skill + Cloud MCP |

**选型原则**：

- **有判断、有顺序 → Skill**：需要 agent 做决策、按步骤执行时
- **有数据、有操作 → MCP**：需要访问外部系统、执行确定性操作时
- **既需要判断又需要数据 → 两者配合**：Skill 调用 MCP 工具是标准模式

---

## 生态质量概览

SkillsBench 评测（[R2]，arXiv 2026）对 47,150 个公开技能的评估：

| 指标 | 数据 |
|------|------|
| 平均质量评分（满分 12） | 6.2 |
| top-quartile（≥9 分）占比 | ~25% |
| 精选技能提升通过率 | 平均 +16.2pp（医疗领域 +51.9）|
| 2–3 个聚焦技能 vs 单一大文档 | +18.6 vs -2.9 |

**关键结论**：质量方差大——仅顶部四分之一的技能有实质提升效果。精选 > 增补。

---

## 选型指南

| 场景 | 推荐工具 | 理由 |
|------|---------|------|
| 教会 agent 一个工作流 | Skill | 按需加载，低 token 开销 |
| 给 agent 访问数据库 | MCP Server | 确定性操作，不占上下文 |
| 快速找到可用技能 | Marketplace | 搜索 description 匹配 |
| 团队共享工具配置 | MCP + Skill 组合 | 数据 + 指令分离 |
| 分发预配置环境 | Plugin（Claude Code）/ Bundle（Hermes）| 打包分发 |

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R1] | <https://skillselion.com/state-of-ai-agent-skills-2026> | The State of AI Agent Skills 2026 | Skillselion 生态追踪：~66K skills、~7.8K MCP、112M 总安装量 |
| [R2] | <https://arxiv.org/abs/2602.12670> | SkillsBench: A Benchmark for Agent Skill Evaluation | 47,150 公开技能平均 6.2/12；精选技能提升 +16.2pp |

[R1]: https://skillselion.com/state-of-ai-agent-skills-2026
[R2]: https://arxiv.org/abs/2602.12670
