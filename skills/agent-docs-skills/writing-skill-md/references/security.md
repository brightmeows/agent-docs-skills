# 技能安全——详细参考

> `writing-skill-md` 参考文件：安全考虑的完整展开。SKILL.md 仅保留摘要与检查清单，详细证据链、注意事项与扩展风险场景见本文件。

## 证据链

技能直接注入代理上下文，恶意或脆弱的技能可导致数据窃取、权限提升等风险。证据链分四层：

- **漏洞发现**：[R4]（USENIX Security 2026）大规模分析发现逾四分之一技能含漏洞，含可执行脚本者风险更高。
- **大规模产业审计**：[R8]（Snyk ToxicSkills，2026-02）扫描 3,984 个技能——**36.82% 含任意级别安全问题、13.4% 达 critical、76 个确认恶意**；规模与时效均优于 R4。
- **可利用性验证**：[R5]（Duan et al., 2026）通过对抗性 prompting 证实真实技能可被利用。
- **供应链投毒**：[R6] 提出 DDIPE——恶意逻辑藏于技能文档的代码示例，代理复用示例时即触发。[R8] 进一步发现 **91% 的恶意技能同时使用 prompt injection + 传统恶意代码**——前者绕过安全机制、后者实施窃取，二者汇聚使传统代码扫描失效。技能已成为新兴软件供应链攻击面。
- **大规模行为审计**：[R10]（Palo Alto Unit 42, 2026-06）引入 Behavioral Integrity Verification（BIV），扫描 49,943 个技能——**80% 存在声明与行为偏差、18.9% 为恶意、2,490 个含多阶段攻击链**。[R11]（Orca Security）发现技能市场中存在全套供应链攻击原语，可组合实现创建→分发→持久化恶意技能的自动化流水线。
- **扫描器绕过实证**：[R12]（CSA, 2026-06）Trail of Bits 研究人员在四小时内开发出三种绕过方法，成功绕过 ClawHub、Cisco 和 Vercel skills.sh 的恶意技能检测器——所有方法均利用已熟知的混淆技术。
- **真实世界攻击验证**：[R13]（AIR, 2026-06）安全公司 AIR 制作了一个虚假技能，使用**可变外部链接**（扫描时指向无害内容，安装后切换 payload）绕过了所有主流市场的安全扫描器，然后通过 Instagram 广告触达 **~26,000 个 agent**（含企业账户）。攻击者可借此完全控制 agent 及其可达的内部系统。
- **行业安全标准建立**：[R14]（OWASP, 2026-03）OWASP 正式发布 **Agentic Skills Top 10（AST10）**
  ——首个面向 agent skill 安全的行业标准框架。详见下方 [OWASP AST10 框架](#owasp-ast10-框架) 小节。

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

### Agent 身份与记忆安全

随着 `SOUL.md`、`MEMORY.md`、`HEARTBEAT.md` 等持久身份文件在 OpenClaw、Hermes Agent、Starpod 等框架中普及，agent 身份层面的攻击面也在扩大：

- **身份投毒**：恶意技能改写 `SOUL.md` 可篡改 agent 的核心身份特征——改变其姓名、角色描述、行为原则。受害者下一会话即按攻击者设定的身份行动，难以察觉（[R8] 已证实此类攻击）
- **记忆文件操纵**：`MEMORY.md` 等长期记忆文件被注入虚假事实后，agent 会在后续会话中持续引用错误信息，形成「认知锚定」效应——代理很难自行质疑记忆文件中的内容
- **跨会话持久化**：记忆投毒的最大危害在于一次写入、长期生效——传统安全扫描只检查单次会话，无法捕获跨会话的影响链
- **身份冒充**：若 agent 框架将 `SOUL.md` 内容作为“不可变身份”对待，恶意技能可利用此信任机制伪装成合法指令源

**应对策略**：

| 措施 | 说明 |
|------|------|
| **只读锁定** | 个人级将 `SOUL.md` 设为只读，禁止技能写入 |
| **写操作显式声明** | 技能如需修改身份/记忆文件，在 `allowed-tools` 和 description 中声明 |
| **定期审计身份文件** | 检查 `SOUL.md`/`MEMORY.md` 内容是否有未授权的变更 |
| **diff 对比** | 更新后对比身份文件前后差异，确认无注入内容 |
| **权限分离** | 利用 OpenCode `permission.skill` 或 Claude Code hook 限制技能对身份文件的写入权限（详见 `structuring-project-agent-md` 机制层参考）|

### 安全扫描器的信任边界

安全扫描器是技能供应链的第一道防线，但**不是最后一道**。[R12] 和 [R13] 连续证实当前主流扫描器存在系统性盲区：

- **可变内容绕过**：扫描时提供无害版本，安装后通过外部 URL 切换为恶意 payload——扫描器的一次性检查无法捕获动态内容
- **混淆技术有效**：所有成功绕过均依赖已熟知的混淆手法（Base64 编码、间接跳转、条件执行），说明扫描器的检测深度有限
- **组合攻击不可见**：单步操作（读文件 + 编码 + 外发）各自看似合法，但组合即构成攻击链。逐条检查无法发现
- **声明-行为不匹配**：技能 description 声明无害功能，但 body 或脚本执行未声明的操作——80% 的技能存在此类偏差 [R10]

**应对策略**：扫描器作为快速过滤层，但不替代人工审查；关注可变外部资源；参考 OWASP Top 10 框架做系统性审计。

### 行为完整性验证

技能声明（description）与实际行为（代码 + 指令）可能不一致——这是技能生态中最容易被忽视的风险。

- **声明≠行为**：[R10] 发现 80% 的技能存在声明与行为偏差。审计技能时至少对照 SKILL.md body 与 description 是否一致
- **多阶段攻击链**：单个无害操作组合可构成攻击链（如 `FILE_READ → base64 → NETWORK_SEND`——读文件、编码、外发，三步各自看似正常，组合即为窃取）。逐条检查难以发现链式攻击
- **指令劫持**：96% 的指令操纵偏差源于恶意意图——攻击者通过自然语言指令而非代码劫持代理决策循环，传统代码扫描无法检测

### 供应链安全

- **技能市场风险**：Skills.sh、ClawHub、claude-plugins.dev 等市场已达 **百万级技能**，但质量参差不齐（平均评分 6.2/12，[R9]）。安装前审阅 SKILL.md 和脚本内容
- **版本锁定**：生产环境使用技能时锁定到 release tag 或 commit SHA，避免 `main` 分支的未审阅变更
- **自动扫描**：部署前用 `mcp-scan` 扫描（`uvx mcp-scan@latest --skills`，[R8]）——把可机器校验的交给机器
- **扫描器有盲区，不单独依赖**：[R12] 已证实主流安全扫描器可被绕过；[R13] 进一步演示了绕过扫描器后触及 26,000 个 agent 的真实攻击。自动扫描是必经检查点，但不是最终安全保证——须结合人工审查
- **关注动态内容**：可变外部链接（扫描时 vs 安装后指向不同内容）是绕过扫描器的主要手法之一。技能引用的外部资源应在安装时验证其静态内容
- **OWASP Top 10 作为审计框架**：参考 [R14] OWASP Agentic Skills Top 10 的十大风险类别逐项审查——覆盖静态扫描难以发现的多阶段攻击链、声明-行为偏差、指令劫持等
- **技能依赖审计**：技能引用的外部工具、MCP 服务器等也应纳入安全审查

---

## OWASP AST10 框架

OWASP Agentic Skills Top 10（[R14]）是首个面向 agent skill 安全的行业标准框架，
将技能生态的主要风险归纳为 10 个类别。以下是核心风险及与编写实践的映射：

| # | 风险 | 严重度 | 核心缓解 | 与本技能编写要求的对应 |
|---|------|--------|---------|----------------------|
| AST01 | **Malicious Skills** | Critical | Merkle 签名 + 注册表扫描 | 锁定版本（tag/commit SHA）；`allowed-tools` 最小权限 |
| AST02 | **Supply Chain Compromise** | Critical | 注册表透明 + 来源追踪 | 审计代码示例与脚本；来源不明技能不自动加载 |
| AST03 | **Over-Privileged Skills** | High | 最小权限声明 + schema 校验 | `allowed-tools` 只给最小工具集；避免通配 `Bash(*)` |
| AST04 | **Insecure Metadata** | High | 静态分析 + 安全解析器 + 沙箱加载 | description 不暴露敏感信息；前端校验禁止 XML 标签 |
| AST05 | **Untrusted External Instructions** | High | 来源清单 + 内容锁定 + 持续重扫 | 可变外部链接风险；部署前校验引用内容 |
| AST06 | **Weak Isolation** | High | 容器化 / Docker 沙箱 | OpenCode 任务 agent / Claude Code subagent 隔离执行 |
| AST07 | **Update Drift** | Medium | 不可变锁定 + 哈希验证 | 生产环境锁定 release tag / commit SHA |
| AST08 | **Poor Scanning** | Medium | 语义 + 行为双通道扫描 | 自动扫描 + 人工审查；不单独依赖扫描结果 |
| AST09 | **No Governance** | Medium | 技能清单 + agent 身份控制 | 团队内建立技能审批流程 |
| AST10 | **Cross-Platform Reuse** | Medium | 通用格式（USF）| 按 agentskills.io 开放标准编写，跨平台兼容 |

### Universal Skill Format（USF）提案

AST10 框架配套提出了 **Universal Skill Format** 提案，
旨在跨平台统一 skill 元数据格式，从源头解决安全元数据丢失问题：

```yaml
---
name: example-skill
version: 1.0.0
platforms: [openclaw, claude, cursor, vscode]
permissions:
  files:
    read: [~/.config/app.json]
    write: [~/.config/app.json]
    deny_write: [SOUL.md, MEMORY.md, AGENTS.md]
  network:
    allow: [api.example.com]
    deny: "*"
  shell: false
  tools: [web_fetch, read_file]
risk_tier: L1
signature: "ed25519:ABCDEF..."
content_hash: "sha256:abcdef..."
---
```

**对编写者的意义**：

- `permissions.deny_write` 保护身份文件（`SOUL.md`/`MEMORY.md`）——当前 `allowed-tools` 不足以表达此粒度
- `network.allow` 是域名白名单而非布尔开关——通配 `network: true` 是 AST03 典型隐患
- `risk_tier` 实现自动化治理策略，无需逐个审查
- USF 尚在提案阶段，但编写技能时已可用 `allowed-tools` + description 声明模拟其理念

---

## 发现即检查清单

部署前额外确认：

- [ ] 无硬编码凭证或密钥
- [ ] `allowed-tools` 未过度授权
- [ ] 脚本文件安全（不执行危险操作）
- [ ] 代码示例与配置模板已审阅（无来源不明的可执行片段）
- [ ] description 不暴露敏感信息
- [ ] description 与实际行为一致——SKILL.md body 不执行 description 未声明的操作
- [ ] 部署前用 `mcp-scan` 或等效工具扫描
- [ ] **扫描器有盲区**——不单独依赖自动扫描结果，结合人工审查确认无可变外部链接/混淆代码
- [ ] 技能若写代理记忆文件（`SOUL.md`/`MEMORY.md` 等），写入内容已审阅
- [ ] **身份文件完整性**：`SOUL.md`/`MEMORY.md` 等身份文件未被技能未授权修改；考虑设为只读
- [ ] 参考 OWASP Agentic Skills Top 10 [R14] 逐项审计（覆盖多阶段攻击链、声明-行为偏差、指令劫持等）
- [ ] 生产环境使用锁定的版本（tag/commit SHA）

---

## 部署后安全治理

技能上线后的安全管理不亚于编写时的安全审查。基于 OWASP AST10 框架（[R14]）与产业最佳实践，建议建立以下治理机制：

### 运行时行为监控

- **技能行为基线**：记录每项技能的典型行为模式（读取了哪些文件、调用了什么工具、网络请求的目标域名），与预期行为对比发现异常
- **异常检测**：监控非预期文件访问（如文本处理技能突然读取 SSH 密钥）、非预期网络外发（如技能 description 未声明网络访问但出现外连）
- **审计日志**：保留技能执行轨迹（调用栈、工具调用序列、读写文件列表），便于事后溯源

### 供应链持续管理

- **版本追踪**：记录每项部署技能的来源（市场/仓库/作者）、版本号、安装时间、content_hash
- **更新审计**：技能更新时对比 diff——不仅看脚本变更，也要审查 SKILL.md body 的指令变更（指令劫持常通过修改 body 而非脚本实现，[R10]）
- **依赖链审查**：技能引用的外部工具、MCP 服务器、运行时库也纳入安全扫描范围
- **废弃技能退役**：不再使用的技能从注册表中移除，避免无人维护的技能成为攻击入口

### 事件响应

- **技能隔离**：发现可疑技能后立即从所有 agent 会话中移除（不依赖代理自身判断，通过工具配置禁用）
- **溯源**：确认事件窗口内受影响的 agent 会话、可能泄露的数据范围
- **修补**：如果是技能本身的漏洞，发布修补版本并强制更新；如果是市场的供应链问题，报告给市场运营方

### 治理成熟度模型

| 等级 | 特征 | 适合场景 |
|------|------|---------|
| **L0 — 无治理** | 不审查、不锁定版本、不记录 | 个人实验、一次性脚本 |
| **L1 — 部署前审查** | 部署前走完发现即检查清单 | 小团队、内部技能 |
| **L2 — 运行时监控** | L1 + 基线对比 + 异常告警 | 中等团队、公开分发的技能 |
| **L3 — 全生命周期** | L2 + 供应链追踪 + 事件响应 + 定期的第三方审计 | 企业级、受监管行业 |

> 治理不是一次性的设置，而是随技能库增长持续演进的实践。从 L1 起步，在首次发现异常行为后升级到 L2。

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R4] | <https://arxiv.org/abs/2601.10338> | Vulnerability Analysis of Agent Skill Ecosystem | 逾四分之一技能含安全漏洞 |
| [R5] | <https://arxiv.org/abs/2604.04989> | SkillAttack: Adversarial Prompting on Agent Skills | 通过对抗性 prompting 可利用技能漏洞 |
| [R6] | <https://arxiv.org/abs/2604.03081> | DDIPE: Supply Chain Poisoning of Agent Skills | 恶意逻辑可藏于代码示例被代理复用 |
| [R8] | <https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/> | ToxicSkills: Agent Skills Supply Chain Audit | 3,984 技能审计：36.82% 含漏洞、91% 恶意技能汇聚 injection+恶意代码、记忆投毒 |
| [R9] | <https://arxiv.org/abs/2602.12670> | SkillsBench: A Benchmark for Agent Skill Evaluation | 87 任务（v1.1）× 11 领域 × 7,308 轨迹；47,150 公开技能平均评分 6.2/12；精选技能提升通过率 +16.2pp |
| [R10] | <https://arxiv.org/abs/2605.11770> | Behavioral Integrity Verification for AI Agent Skills | Unit 42 BIV：49,943 技能中 80% 有行为偏差、18.9% 恶意、2,490 个含多阶段攻击链 |
| [R11] | <https://orca.security/resources/blog/ai-agent-skill-supply-chain-security/> | AI Agent Skill Supply Chain Attack Vectors | Orca Security 发现技能市场中全套供应链攻击原语 |
| [R12] | <https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/06/CSA_research_note_AI_agent_skill_scanner_bypass_20260610-csa-styled.pdf> | AI Agent Skill Scanner Bypass | CSA 证实技能安全扫描器可被绕过 |
| [R13] | <https://www.air.security/blog-posts/the-story-of-skills> | The Story of Skills — How We Hijacked 26,000 Agents | AIR 证实虚假技能可绕过所有扫描器，触及 26,000 agent，含企业账户 |
| [R14] | <https://owasp.org/www-project-agentic-skills-top-10/> | OWASP Agentic Skills Top 10 | 首个 agent skill 安全行业标准框架：10 类风险、Universal Skill Format 提案、跨平台兼容方案 |
| [R15] | <https://github.com/OWASP/www-project-agentic-skills-top-10/blob/main/docs/OWASP-Agentic-Skills-Top10-v0.5.pdf> | OWASP AST10 Full Report (v0.5) | 完整 PDF 报告含十大风险详情、攻击场景、缓解措施 |

## 本地参考

| 编号 | 文件 | 用途 |
| [L1] | [writing-skill-md SKILL.md](../SKILL.md) | 主技能文件，安全摘要与检查清单 |
