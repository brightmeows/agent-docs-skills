# 技能安全——详细参考

> `writing-skill-md` 参考文件：安全考虑的完整展开。SKILL.md 仅保留摘要与检查清单，详细证据链、注意事项与扩展风险场景见本文件。

## 证据链

技能直接注入代理上下文，恶意或脆弱的技能可导致数据窃取、权限提升等风险。证据链分四层：

- **漏洞发现**：[R4]（USENIX Security 2026）大规模分析发现逾四分之一技能含漏洞，含可执行脚本者风险更高。
- **大规模产业审计**：[R8]（Snyk ToxicSkills，2026-02）扫描 3,984 个技能——**36.82% 含任意级别安全问题、13.4% 达 critical、76 个确认恶意**；规模与时效均优于 R4。
- **可利用性验证**：[R5]（Duan et al., 2026）通过对抗性 prompting 证实真实技能可被利用。
- **供应链投毒**：[R6] 提出 DDIPE——恶意逻辑藏于技能文档的代码示例，代理复用示例时即触发。[R8] 进一步发现 **91% 的恶意技能同时使用 prompt injection + 传统恶意代码**——前者绕过安全机制、后者实施窃取，二者汇聚使传统代码扫描失效。技能已成为新兴软件供应链攻击面。
- **大规模行为审计**：[R10]（Palo Alto Unit 42, 2026-06）引入 Behavioral Integrity Verification（BIV），扫描 49,943 个技能——**80% 存在声明与行为偏差、18.9% 为恶意、2,490 个含多阶段攻击链**。[R11]（Orca Security）发现技能市场中存在全套供应链攻击原语，可组合实现创建→分发→持久化恶意技能的自动化流水线。[R12]（CSA）证实技能安全扫描器本身可被绕过。

## 编写安全注意事项

### 凭证与密钥

- **避免在脚本中硬编码凭证**——API key、token 等不应包含在技能脚本或参考文件中
- **description 不暴露敏感信息**——技能描述注入系统提示词，不应包含内部路径、凭证或密钥
- **环境变量替代硬编码**：需要认证的技能应指导代理从环境变量或密钥管理服务读取凭证

### 最小权限原则

- **`allowed-tools` 遵循最小权限原则**——只给技能完成任务所需的最小工具集，避免开放 `Bash(*)`、`Read(*)` 等通配权限
- **OpenCode 权限模式**：在 `opencode.json` 中用 `permission.skill` 定义技能级别的 allow/deny/ask（详见 `structuring-project-agent-md` 的 OpenCode 参考）
- **Claude Code hooks 兜底**：`PreToolUse` hook 可确定性阻断危险命令（exit 2），与技能 `allowed-tools` 形成纵深防御

### 代码示例与脚本

- **公开分发的技能需审计脚本**——`scripts/` 目录下的可执行文件可能被代理在用户环境中运行，必须确保无害
- **审慎对待代码示例**——技能中的代码示例与配置模板会被代理复用执行（DDIPE 攻击载体，[R6]）；借鉴第三方示例时先审阅其完整逻辑，避免照搬来源不明的片段
- **来源不明技能不自动加载**——来自不可信源的技能应先审阅 SKILL.md 和脚本再启用
- **提供预制脚本**优先于让代理现场生成——预制脚本可控、可审计、可签名

### 记忆持久化风险

- **审慎对待会修改记忆/状态文件的技能**——[R8] 发现恶意技能可改写代理记忆文件（如 `SOUL.md`、`MEMORY.md`）实现跨会话持久化投毒；审查技能是否写记忆文件、写入内容是否可信
- **记忆文件写操作应显式声明**：如果技能需要写入记忆文件，应在 SKILL.md 中说明写入内容与原因

### 行为完整性验证

技能声明（description）与实际行为（代码 + 指令）可能不一致——这是技能生态中最容易被忽视的风险。

- **声明≠行为**：[R10] 发现 80% 的技能存在声明与行为偏差。审计技能时至少对照 SKILL.md body 与 description 是否一致
- **多阶段攻击链**：单个无害操作组合可构成攻击链（如 `FILE_READ → base64 → NETWORK_SEND`——读文件、编码、外发，三步各自看似正常，组合即为窃取）。逐条检查难以发现链式攻击
- **指令劫持**：96% 的指令操纵偏差源于恶意意图——攻击者通过自然语言指令而非代码劫持代理决策循环，传统代码扫描无法检测

### 供应链安全

- **技能市场风险**：Skills.sh、ClawHub、claude-plugins.dev 等市场已达 **百万级技能**，但质量参差不齐（平均评分 6.2/12，[R9]）。安装前审阅 SKILL.md 和脚本内容
- **版本锁定**：生产环境使用技能时锁定到 release tag 或 commit SHA，避免 `main` 分支的未审阅变更
- **自动扫描**：部署前用 `mcp-scan` 扫描（`uvx mcp-scan@latest --skills`，[R8]）——把可机器校验的交给机器
- **技能依赖审计**：技能引用的外部工具、MCP 服务器等也应纳入安全审查

## 发现即检查清单

部署前额外确认：

- [ ] 无硬编码凭证或密钥
- [ ] `allowed-tools` 未过度授权
- [ ] 脚本文件安全（不执行危险操作）
- [ ] 代码示例与配置模板已审阅（无来源不明的可执行片段）
- [ ] description 不暴露敏感信息
- [ ] description 与实际行为一致——SKILL.md body 不执行 description 未声明的操作
- [ ] 部署前用 `mcp-scan` 或等效工具扫描
- [ ] 技能若写代理记忆文件（`SOUL.md`/`MEMORY.md` 等），写入内容已审阅
- [ ] 生产环境使用锁定的版本（tag/commit SHA）

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R4] | [arXiv:2601.10338](https://arxiv.org/abs/2601.10338) | Vulnerability Analysis of Agent Skill Ecosystem | 逾四分之一技能含安全漏洞 |
| [R5] | [arXiv:2604.04989](https://arxiv.org/abs/2604.04989) | SkillAttack: Adversarial Prompting on Agent Skills | 通过对抗性 prompting 可利用技能漏洞 |
| [R6] | [arXiv:2604.03081](https://arxiv.org/abs/2604.03081) | DDIPE: Supply Chain Poisoning of Agent Skills | 恶意逻辑可藏于代码示例被代理复用 |
| [R8] | [snyk.io](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) | ToxicSkills: Agent Skills Supply Chain Audit | 3,984 技能审计：36.82% 含漏洞、91% 恶意技能汇聚 injection+恶意代码、记忆投毒 |
| [R9] | [arXiv:2602.12670](https://arxiv.org/abs/2602.12670) | SkillsBench: A Benchmark for Agent Skill Evaluation | 84 任务 × 11 领域 × 7,308 轨迹；47,150 公开技能平均评分 6.2/12；精选技能提升通过率 +16.2pp |
| [R10] | [arXiv:2605.11770](https://arxiv.org/abs/2605.11770) | Behavioral Integrity Verification for AI Agent Skills | Unit 42 BIV：49,943 技能中 80% 有行为偏差、18.9% 恶意、2,490 个含多阶段攻击链 |
| [R11] | [orca.security](https://orca.security/resources/blog/ai-agent-skill-supply-chain-security/) | AI Agent Skill Supply Chain Attack Vectors | Orca Security 发现技能市场中全套供应链攻击原语 |
| [R12] | [cloudsecurityalliance.org](https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/06/CSA_research_note_AI_agent_skill_scanner_bypass_20260610-csa-styled.pdf) | AI Agent Skill Scanner Bypass | CSA 证实技能安全扫描器可被绕过 |

## 本地参考

| 编号 | 文件 | 用途 |
| [L1] | [writing-skill-md SKILL.md](../SKILL.md) | 主技能文件，安全摘要与检查清单 |
