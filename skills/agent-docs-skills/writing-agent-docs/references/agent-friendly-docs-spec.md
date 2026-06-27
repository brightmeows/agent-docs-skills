# Agent-Friendly Documentation Spec

> `writing-agent-docs` 参考文件：Agent-Friendly Documentation Spec（[agentdocsspec.com](https://agentdocsspec.com/spec/)）的概要与映射。
> 本文件是**文档站点面向 agent 的技术基础设施层标准**，与 `writing-agent-docs` 的内容写作原则互补——前者管"文档怎么写"，后者管"站点怎么为 agent 提供文档"。完整规范见 [R1]。

---

## 定位

Agent-Friendly Documentation Spec（草案 v0.5.1，2026-05-08）定义了评估文档站点对 coding agent 友好程度的 23 项检查，分 7 大类别。它解决的是**基础设施层**问题——即使内容写得再好，如果站点不兼容 agent 的 web fetch 管道，代理也无法有效消费文档。

本仓库的写作原则（`writing-agent-docs`）管**内容层**（写什么、怎么写、怎么组织），Agent-Friendly Documentation Spec 管**交付层**（内容怎么被 agent 获取）。两者互补：

| 层面 | 解决的问题 | 对应标准 |
|------|-----------|---------|
| **内容层** | 文档该写什么、怎么写、怎么组织 | 本仓库（writing-agent-docs & 子技能）|
| **交付层** | 文档站点如何被 agent 有效获取 | Agent-Friendly Documentation Spec |
| **发现层** | agent 如何找到适合的技能/文档 | SKILL.md CSO / `llms.txt` / `.well-known/` |

---

## 23 项检查概览

### Category 1：内容可发现性（Content Discoverability）

| 检查 | 内容 | 核心建议 |
|------|------|---------|
| `llms-txt-exists` | 站点是否提供 `llms.txt` | 在站点根目录提供 `llms.txt`，这是影响最大的单项改进 |
| `llms-txt-valid` | `llms.txt` 是否符合建议结构 | H1 标题 + blockquote 摘要 + H2 章节 + markdown 链接列表 |
| `llms-txt-links-resolve` | `llms.txt` 中链接是否可解析 | 失效链接比没有 `llms.txt` 更糟 |
| `llms-txt-size` | `llms.txt` 是否超过截断阈值 | 目标 <50,000 字符；超限则用嵌套模式分拆 |
| `llms-txt-links-markdown` | 链接是否指向 markdown 内容 | 优先指向 `.md` URL 而非 HTML |
| `llms-txt-directive-html` | HTML 页面是否含指向 `llms.txt` 的指令 | 在每个页面顶部添加 agent 可见的 `llms.txt` 引导 |
| `llms-txt-directive-md` | Markdown 页面是否含指向 `llms.txt` 的指令 | 与 HTML 同理 |

### Category 2：Markdown 可用性（Markdown Availability）

| 检查 | 核心建议 |
|------|---------|
| `markdown-url-support` | 在 URL 后追加 `.md` 返回 markdown 内容 |
| `content-negotiation` | 响应 `Accept: text/markdown` 返回 clean markdown |

### Category 3：页面大小与截断风险（Page Size & Truncation）

| 检查 | 核心建议 |
|------|---------|
| `rendering-strategy` | 避免客户端渲染（SSR/SSG 确保内容在 HTTP 响应中）|
| `page-size-markdown` | Markdown 版本 <50,000 字符 |
| `page-size-html` | HTML 转换后 <100,000 字符 |
| `content-start-position` | 正文在转换输出前 10% 内开始 |

### Category 4：内容结构（Content Structure）

| 检查 | 核心建议 |
|------|---------|
| `tabbed-content-serialization` | Tab 内容序列化后 <50,000 字符 |
| `section-header-quality` | Tab 内容的标题应含语境区分（如"步骤 1（Python）"）|
| `markdown-code-fence-validity` | 代码 fence 必须成对闭合 |

### Category 5–7：URL 稳定性、认证与缓存

含 HTTP 状态码、重定向行为、认证网关检测、缓存头卫生等。

> 完整规范定义见 [R1]。

---

## 与本仓库写作原则的映射

| Agent-Friendly Spec 原则 | 本仓库对应 |
|-------------------------|-----------|
| `llms.txt` 作为导航入口 | `writing-skill-md` CSO（技能发现机制）|
| Markdown 优先于 HTML | `writing-agent-docs` B.1 简洁优先——尽量以可直接执行的指令呈现 |
| 页面避免截断（<50K 字符）| `writing-skill-md` body <500 行 / 指令 <5K tokens |
| Tab 内容序列化风险 | `writing-agent-docs` C.4 结构化优先——表格比 tab 更 agent-friendly |
| 渐进式披露大文档集 | `writing-agent-docs` C.2 渐进式披露——重内容下沉到子文件 |
| 声明-行为一致性 | `writing-skill-md` 安全参考 OWASP AST10——description 必须与 body 一致 |

---

## 关键区别

**Agent-Friendly Documentation Spec 不是 AGENTS.md/SKILL.md 的替代品。** 它关心的是：

- 一个从 URL 获取文档页面的 agent 能否获得可用的内容
- 页面是否因截断、客户端渲染、tab 序列化而丢失信息
- 站点是否提供 agent 可发现的内容索引（`llms.txt`）

而本仓库关心的是：

- 文档内容本身是否质量高、token 高效、可遵从
- 配置（AGENTS.md）和技能（SKILL.md）的写作方法与格式

两者解决不同层次的问题，一起用效果最佳。

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R1] | <https://agentdocsspec.com/spec/> | Agent-Friendly Documentation Spec v0.5.1 | 23 项检查 × 7 类别，评估文档站点对 coding agent 的友好程度。含 `llms.txt`、markdown 可用性、截断风险、内容结构、URL 稳定性等 |

[R1]: https://agentdocsspec.com/spec/
