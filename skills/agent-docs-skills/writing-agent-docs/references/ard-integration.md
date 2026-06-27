# ARD（Agentic Resource Discovery）参考

> `writing-agent-docs` 参考文件。
> ARD 是 2026 年 6 月由 Google、Microsoft、Hugging Face 等联合发布的开放规范（v0.9 草案），
> 定义 agent 资源（技能、MCP 服务器、agent 自身）的发布、发现与验证协议。
> 正式规范见 [agenticresourcediscovery.org](https://agenticresourcediscovery.org)，GitHub 仓库
> [ards-project/ard-spec](https://github.com/ards-project/ard-spec)。

---

## 定位

ARD 是关于 **agent 资源发现层** 的规范。它与 `.well-known/agent-skills/index.json` 互补而非竞争：

| 维度 | `.well-known/index.json`（agentskills.io） | ARD |
|------|------------------------------------------|-----|
| **覆盖范围** | 仅 Agent Skills | Skills + MCP Servers + Agents + API |
| **发现方式** | 约定路径 `.well-known/agent-skills/` | HTTP `Link` header / `/.well-known/ai-catalog.json` / `robots.txt` Agentmap / DNS |
| **托管位置** | Git 仓库根目录 | 任意 Web 服务器 |
| **验证机制** | digest SHA256 | 签名 + 元数据校验（trustManifest） |
| **成熟度** | 事实标准，广泛采用 | 草案阶段（2026-06 发布，v0.9） |
| **治理** | AAIF / Linux Foundation | ARD Working Group（Google/Microsoft/Hugging Face 等）|

**场景对照：**

| 场景 | 用哪个 |
|------|--------|
| 仓库内技能发现 | `.well-known/agent-skills/index.json`（所有市场均支持）|
| 跨站点技能/工具/agent 发现 | ARD（HTTP 可发现）|
| 生产环境技能供应链 | `.well-known/` + ARD 双通道 |
| 个人/团队技能库 | ARD（可托管于个人站点）|
| 企业合规环境 | ARD（trustManifest + 签名校验）|

---

## 核心概念

### 问题

AGENTS.md / SKILL.md 解决仓库级别的代理配置问题。但当 agent 需要跨越组织边界发现可用资源时——"有没有一个支付处理的 MCP 服务器？"——需要一个标准化的发现协议。ARD 正是为此而生。

### 架构

ARD 定义两个核心原语：

1. **Catalog（目录）**：组织在其域名下发布 `ai-catalog.json`，描述其可用资源。域名即身份锚点。
2. **Registry（注册中心）**：充当 agent 资源的搜索引擎——爬取 catalog、建立索引、提供搜索接口。

### 设计原则

- **搜索优先**：动态发现而非预安装（类比搜索引擎而非应用商店）
- **上下文窗口外扩展**：发现移出 LLM 上下文到专用搜索服务
- **工件无关信封**：不约束内部 schema，仅用 `type` 字段标识资源类型
- **严格值或引用**：`url` 和 `data` 二选一，互斥
- **REST 基线**：所有实现必须暴露 HTTP REST 搜索接口
- **关注点分离**：认证委托给工件协议，分发属于基础设施层

---

## 核心数据模型：ai-catalog.json

资源清单文件，托管在 `/.well-known/ai-catalog.json`。

### 文件结构

```json
{
  "specVersion": "1.0",
  "host": {
    "displayName": "Acme Enterprise AI",
    "identifier": "did:web:acme.com"
  },
  "entries": [
    {
      "identifier": "urn:air:acme.com:agent:assistant",
      "displayName": "Corporate Assistant (A2A)",
      "type": "application/a2a-agent-card+json",
      "url": "https://api.acme.com/agents/assistant.json",
      "description": "General-purpose corporate A2A assistant.",
      "representativeQueries": [
        "help me draft an email to the security working group",
        "summarize my unread messages from Todd"
      ]
    }
  ]
}
```

### 核心字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `identifier` | String | 全局唯一 ID，格式 `urn:air:<publisher>:<namespace>:<agent-name>` |
| `displayName` | String | 人类可读名称 |
| `type` | String | IANA Media Type（如 `application/mcp-server-card+json`）|

**必须二选一：**

| 字段 | 说明 |
|------|------|
| `url` | 远程引用资源文档 |
| `data` | 内联嵌入完整资源文档 |

**可选字段：**

| 字段 | 说明 |
|------|------|
| `description` | 简短描述 |
| `tags` | 关键词过滤 |
| `capabilities` | 能力列表，用于快速过滤 |
| `representativeQueries` | 2–5 条自然语言查询示例，用于语义排序 |
| `version` | 资源版本 |
| `trustManifest` | 可验证身份和信任元数据 |

### 资源标识符格式

```
urn:air:<publisher>:<namespace>:<agent-name>
```

- `urn:air:` — 固定前缀，AI Artifact Resource 命名空间
- `<publisher>` — 域名（如 `acme.com`），即信任锚点
- `<namespace>` — 可选层级分类（如 `finance:trading`）
- `<agent-name>` — 资源短名称（如 `assistant`）

设计理由：将逻辑标识与物理位置解耦（HTTP URL 随部署变动而断裂），
同时通过域名锚定实现去中心化信任验证。

---

## 发现机制

发布者通过以下方式宣告资源清单：

| 机制 | 方式 |
|------|------|
| **Well-Known URI** | `https://{domain}/.well-known/ai-catalog.json` |
| **Agentmap** | `robots.txt` 中添加 `Agentmap: https://example.com/catalog.json` |
| **HTML Link Tag** | `<link rel="ai-catalog" href="/.well-known/ai-catalog.json">` |
| **DNS** | `_catalog._agents.example.com` 或 `_search._agents.example.com` 的 Service Binding 记录 |

---

## 注册中心 API

ARD Agent Registry **必须**暴露标准的 HTTP REST 搜索接口。

### 搜索（POST /search）— 必须

自然语言查询 + 结构化过滤，返回排序结果。

```json
{
  "query": {
    "text": "find me a flight booking agent",
    "filter": {
      "type": ["application/a2a-agent-card+json"]
    }
  },
  "federation": "referrals",
  "pageSize": 5
}
```

### 探索（POST /explore）— 可选

返回聚合统计而非排序结果——用于内省注册中心容量。

### 列表（GET /agents）— 可选

确定性浏览，适用于开发者门户，可缓存。

### 联邦（Federation）

| 模式 | 行为 |
|------|------|
| `auto` | 自动查询上游注册中心，返回合并结果 |
| `referrals` | 返回自身结果 + 推荐的其他注册中心 |
| `none` | 仅搜索自身索引 |

---

## 与本仓库的协同

| 本仓库组件 | 与 ARD 的关系 |
|-----------|--------------|
| `.well-known/agent-skills/index.json` | 技能清单的子集——ARD 可引用或包含此文件 |
| `SKILL.md` | ARD 的 `manifest` 可指向 SKILL.md URL |
| `AGENTS.md` | AGENTS.md 描述项目范围，ARD 描述可用资源——互补 |
| MCP Server 配置 | ARD 可注册 MCP Server，统一发现入口 |

### 三层 Agent 发现栈

ARD 与 `AGENTS.md`、`llms.txt` 构成互补的三层发现栈（[R1]）：

| 层 | 标准 | 文件 | 触发时机 | 作用 |
|----|------|------|---------|------|
| 环境层 | AGENTS.md | 仓库根目录 | 会话开始（最先触发）| 注入项目偏好，翻转代理选择 |
| 检索层 | llms.txt | `/llms.txt` | 代理读取文档时 | 引导代理正确读取文档 |
| 注册层 | ARD | `/.well-known/ai-catalog.json` | 代理查询注册中心时 | 让资源和工具有被发现性 |

三者不重叠、不竞争——它们在不同阶段解决不同问题。对于开发者工具，
`AGENTS.md` 杠杆最高（100% 选择翻转效果 [R1]）；对于企业级 API，
ARD 是必备入口。

---

## 身份与信任

ARD 通过可选的 `trustManifest` 对象提供企业级信任验证：

| 字段 | 类型 | 说明 |
|------|------|------|
| `identity` | String | SPIFFE ID / DID / HTTPS URI |
| `identityType` | String | 类型提示（"did"、"spiffe"、"https"）|
| `attestations` | Array | 合规认证列表（SOC2、HIPAA、GDPR 等）|
| `provenance` | Array | 溯源记录 |
| `signature` | String | 对 trustManifest 内容的 detached JWS 签名 |

信任验证域与域名锚定一致：`urn:air:acme.com:*` 的条目必须能提供
`acme.com` 签名的加密凭证。

---

## 快速集成

### 部署步骤

1. 准备资源清单 JSON（遵循 [ARD schema](https://github.com/ards-project/ard-spec/blob/main/spec/schemas/ai-catalog.schema.json)）
2. 托管到 Web 服务器，路径 `/.well-known/ai-catalog.json`
3. 或在 HTTP 响应头加 `Link: <https://example.com/.well-known/ai-catalog.json>; rel="ai-catalog"`
4. 或在 `robots.txt` 加 `Agentmap: https://example.com/.well-known/ai-catalog.json`
5. 验证：`curl -s https://example.com/.well-known/ai-catalog.json | jq '.entries | length'`
6. （可选）使用官方一致性测试工具：

   ```bash
   git clone https://github.com/ards-project/ard-spec.git
   ./ard-spec/conformance/bin/conformance-test manifest https://example.com/.well-known/ai-catalog.json
   ```

### 最简单路径（Solo Developer）

对于个人开发者，无需复杂身份体系。只需在 GitHub Pages 上托管一个
`ai-catalog.json`：

```json
{
  "specVersion": "1.0",
  "host": { "displayName": "Alice's AI Tools" },
  "entries": [
    {
      "identifier": "urn:air:github.com:alice-dev:pptx-creator",
      "displayName": "pptx-creator",
      "type": "application/ai-skill",
      "url": "https://github.com/alice-dev/pptx-creator",
      "description": "Create professional PowerPoint presentations following brand guidelines."
    }
  ]
}
```

---

## 注意事项

- **成熟度风险**：ARD v0.9 尚在草案阶段（2026-06），规范可能变动。生产环境优先用 `.well-known/`，ARD 作扩展通道
- **验证优先**：`integrity` 字段使用 subresource integrity（SRI）格式，部署前校验资源哈希
- **`representativeQueries` 是关键**：2–5 条精确的任务导向查询短语决定资源是否能被正确发现——思考"代理会如何搜索这个工具"
- **权限声明**：ARD 资源清单不含权限字段。权限声明仍通过 SKILL.md 的 `allowed-tools` 和 `permission` 机制
- **命名不可变**：`identifier` 一旦发布不可更改（指向逻辑身份），物理位置变更通过更新 `url` 处理

---

## 本地参考

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [ecosystem.md](./ecosystem.md) | 生态全景：Agent Skills / MCP / Marketplaces 三大类工具 |

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R1] | <https://www.synscribe.com/blog/ard-vs-llms-txt-vs-agents-md-comparison> | ARD vs. llms.txt vs. AGENTS.md: Which Agentic Discovery Standard Do You Actually Need? | 三层发现栈对比——环境层/检索层/注册层；AGENTS.md 100% 选择翻转效果 |
| [R2] | <https://developers.googleblog.com/announcing-the-agentic-resource-discovery-specification/> | Announcing the Agentic Resource Discovery Specification | Google 官方发布公告——架构、设计动机、Gemini Enterprise 集成计划 |
| [R3] | <https://agenticresourcediscovery.org/spec/> | ARD Specification v0.9 | 正式规范——数据模型、API 定义、联邦机制、信任验证、Schema |
