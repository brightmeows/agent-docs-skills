# 生态全景——Agent Skills / MCP / Marketplaces

> `writing-agent-docs` 参考文件：技能生态的完整参考——生态构成、职责划分、市场类型、组合模式、质量验证、发布指南与选型。
> 合并自原 `ecosystem-landscape.md` 与 `writing-skill-md` 的 `ecosystem-publishing.md`，作为生态知识单一真理源。

---

## 生态构成

公开追踪的 AI agent 工具生态由三大类构成：

| 类型 | 职责 |
|------|------|
| **Agent Skills** | 指令与工作流（SKILL.md），教会 agent **做什么** |
| **MCP Servers** | 外部工具与数据连接，给 agent **用什么** |
| **Marketplaces** | 分发、发现与安全扫描 |

> 生态规模与精确数据见参考表 [R1]、[R3]。质量参差——公开技能平均评分偏低，仅顶部四分之一有实质提升（[R2]）。
>
> **代码级补充**：代码内联文档标准（如 SAGE Spec——[sage-spec](https://github.com/mikewcasale/sage-spec)）在 docstring 中用 `@graph`、`@agent-guidance` 等标签嵌入 agent 指导。这是与 Skills/MCP/Marketplaces 不同层次的标准——解决"代码本身如何为 agent 提供上下文"的问题，与本仓库的配置文件级指导互补。

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
- **典型市场**：见下方[技能市场与注册中心](#技能市场与注册中心)

---

## 技能市场与注册中心

技能生态已具规模——SKILL.md 是分发代理知识的标准载体，而非小众实验。
质量参差——精选技能有效，平均水平偏低（[R2]；精确数据见参考表 [R1]、[R3]）。

| 市场类型 | 特点 | 代表平台 |
|---------|------|---------|
| **CLI 安装型** | `npx skills install`、Snyk 安全扫描、策展推荐 | Vercel Skills.sh |
| **社区索引型** | 自动索引 GitHub 公开 SKILL.md，附质量指标 | ClawHub |
| **插件集合型** | 按工具分类聚合技能，开源社区维护 | claude-plugins.dev |
| **全网抓取型** | 大规模 GitHub 扫描，覆盖面广、但无审查 | SkillsMP |

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

## 生态质量验证

SkillsBench（[R2]）是首个 peer-reviewed 技能评估基准，基于 87 个任务 × 11 个领域 × 7,308 条轨迹。对数万个公开技能的评估：

| 指标 | 数据 |
|------|------|
| 平均质量评分（满分 12） | 6.2 |
| top-quartile（≥9 分）占比 | ~25% |
| 精选技能提升通过率 | 平均 +16.2pp（医疗领域 +51.9）|
| 2–3 个聚焦技能 vs 单一大文档 | +18.6 vs -2.9 |

**关键结论**：质量方差大——仅顶部四分之一的技能有实质提升效果。精选 > 增补。

---

## 发布技能

1. **遵循开放标准**：确保 SKILL.md 格式符合 [agentskills.io](https://agentskills.io/specification) 规范——所有市场均基于同一标准
2. **版本控制**：使用 Git tag 管理版本，发布时锁定到 release tag（`npx skills add <url>#v1.0.0`）
3. **GitHub 公开仓库**：将技能放在公开 GitHub 仓库的 `skills/` 目录下，市场将自动索引
4. **安全扫描**：发布前用 `mcp-scan` 扫描（`uvx mcp-scan@latest --skills`）
5. **description 优化**：按 CSO 原则编写 description，确保市场搜索能匹配到你的技能

---

## 跨工具兼容性

SKILL.md 是开放标准——跨工具兼容是其设计目标，写一次即可跨平台使用。
完整兼容列表见 [agentskills.io 展示页](https://agentskills.io/clients)。
部分工具额外支持 Claude Code 扩展字段的子集。

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
| [R1] | <https://skillselion.com/state-of-ai-agent-skills-2026> | The State of AI Agent Skills 2026 | Skillselion 生态追踪：84K+ 工具（65K skills、7.8K MCP、8.3K 市场）、112M 总安装量 |
| [R2] | <https://arxiv.org/abs/2602.12670> | SkillsBench: A Benchmark for Agent Skill Evaluation | 47,150 公开技能平均 6.2/12；精选技能提升 +16.2pp |
| [R3] | <https://agentman.ai/blog/agent-skills-ecosystem-report-2026> | The Agent Skills Ecosystem in 2026 | 2026-06 报告：~40 兼容产品、1.9M+ 公开技能、22,511 技能安全审计、质量/安全双维度警示 |
