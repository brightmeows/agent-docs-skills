# ARD（Agentic Resource Discovery）集成指南

> `structuring-project-agent-md` 参考文件。
> ARD 是 2026 年 6 月发布的开放规范（Google 联合 Microsoft、Hugging Face 等），定义 agent 资源（技能、MCP 服务器、agent 自身）的发布、发现与验证协议。
> 正式规范见 [agenticresourcediscovery.org](https://agenticresourcediscovery.org)，GitHub 仓库 [ards-project/ard-spec](https://github.com/ards-project/ard-spec)。

---

## 定位

ARD 与 `.well-known/agent-skills/index.json` 互补而非竞争：

| 维度 | `.well-known/index.json`（agentskills.io） | ARD |
|------|------------------------------------------|-----|
| **覆盖范围** | 仅 Agent Skills | Skills + MCP Servers + Agents |
| **发现方式** | 约定路径 `.well-known/agent-skills/` | HTTP `Link` header / `/.well-known/ard` |
| **托管位置** | Git 仓库根目录 | 任意 Web 服务器 |
| **验证机制** | digest SHA256 | 签名 + 元数据校验 |
| **成熟度** | 事实标准，广泛采用 | 草案阶段（2026-06 发布） |

**何时用哪个：**

| 场景 | 用哪个 |
|------|--------|
| 仓库内技能发现 | `.well-known/agent-skills/index.json`（所有市场均支持）|
| 跨站点技能/工具/agent 发现 | ARD（HTTP 可发现）|
| 生产环境技能供应链 | `.well-known/` + ARD 双通道 |
| 个人/团队技能库 | ARD（可托管于个人站点）|

## 快速集成

```json
{
  "@context": "https://agenticresourcediscovery.org/context.json",
  "resources": [
    {
      "type": "Skill",
      "id": "https://example.com/skills/code-review",
      "name": "code-review",
      "description": "审查 PR 代码质量与安全性",
      "manifest": "https://example.com/skills/code-review/SKILL.md",
      "integrity": "sha256:abc123..."
    },
    {
      "type": "McpServer",
      "id": "https://example.com/mcp/db-query",
      "name": "db-query",
      "description": "PostgreSQL 查询接口",
      "manifest": "https://example.com/mcp/db-query/manifest.json"
    }
  ]
}
```

### 部署步骤

1. 准备资源清单 JSON（遵循 ARD schema）
2. 托管到 Web 服务器，路径约定为 `/.well-known/ard`
3. 或在 HTTP 响应头加 `Link: <https://example.com/ard.json>; rel="agent-resource-discovery"`
4. 验证：`curl -s https://example.com/.well-known/ard | jq '.resources | length'`

## 与本仓库的协同

| 本仓库组件 | 与 ARD 的关系 |
|-----------|--------------|
| `.well-known/agent-skills/index.json` | 技能清单的子集——ARD 可引用或包含此文件 |
| `SKILL.md` | ARD 的 `manifest` 可指向 SKILL.md URL |
| `AGENTS.md` | AGENTS.md 描述项目范围，ARD 描述可用资源——互补 |
| MCP Server 配置 | ARD 可注册 MCP Server，统一发现入口 |

## 注意事项

- **成熟度风险**：ARD 尚在草案阶段（2026-06），规范可能变动。生产环境优先用 `.well-known/`，ARD 作扩展通道
- **验证优先**：`integrity` 字段使用 subresource integrity（SRI）格式，部署前校验资源哈希
- **权限声明**：ARD 资源清单不含权限字段。权限声明仍通过 SKILL.md 的 `allowed-tools` 和 `permission` 机制

## 本地参考

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [cross-tool-compat.md](./cross-tool-compat.md) | ARD 与 `.well-known/` 异同对比（SKILL.md 基表）|
