# 技能全生命周期管理

> `writing-skill-md` 参考文件：技能从创建到废弃的完整管理流程。
> 主 SKILL.md 的技能创建清单、验证与自检、生态与发布覆盖创建与发布阶段；
> 本文件补充更新、版本策略、废弃等后续阶段。

---

## 阶段总览

| 阶段 | 核心产出 | 验证标准 |
|------|---------|---------|
| **创建** | SKILL.md + 目录结构 | 技能创建清单（主 SKILL.md）|
| **测试** | 验证通过的技能 | 按类型的验证强度（主 SKILL.md）|
| **发布** | 可安装的技能 | `mcp-scan` 通过 + description 检查 |
| **更新** | 新版本 | diff 审查 + 兼容性声明 |
| **废弃** | 废弃标记 | 替代方案指引 + 移除计划 |

---

## 版本策略

技能版本与代码版本独立管理，遵循简化 SemVer：

| 版本号变更 | 何时用 | 示例 |
|-----------|--------|------|
| **主版本** | 不向后兼容的指令/结构变更 | 重写全部描述、移除字段 |
| **次版本** | 新增章节/功能、非破坏性修改 | 新增常见错误节、补充示例 |
| **补丁** | 拼写修正、引用更新、格式化 | 修复失效链接、更新 digest |

### 版本号存放

```yaml
---
name: my-skill
version: 1.2.3    # 次版本 + 补丁用于小修改；主版本用于破坏性变更
---
```

- 版本号放在 SKILL.md 的 `version` 字段（可选字段，提案中——见 [R13]）
- 未采纳前，版本信息记录在 git tag 中：`git tag <skill-name>/v1.0.0`
- 发布到市场时用 git tag 锁定版本：`npx skills add <url>#<skill-name>/v1.0.0`

---

## 更新流程

### 更新前检查

| 检查项 | 说明 |
|--------|------|
| 变更是否有真实需求驱动 | 非推测性——有用户反馈或观测到的问题 |
| 现有用户不受破坏性影响 | 主版本变更需在 description 或 CHANGELOG 中标注 |
| description 是否需要同步更新 | 功能变更后 description 须一致（防声明-行为偏差）|
| digest 已更新 | `.well-known/agent-skills/index.json` 的 digest 须匹配 |
| 安全审计覆盖新内容 | 新加脚本/代码示例须走审计流程 |

### 更新步骤

1. 在 SKILL.md 中标注变更（可加 `> 自 vX.Y 变更` 标注）
2. 更新参考文件的版本引用（如有）
3. 重新计算 digest 并更新 `.well-known/agent-skills/index.json`
4. 版本 tag：`git tag <skill-name>/v<新版本>`
5. 发布时在 release note 中列明变更类型（新增/修正/废弃）

---

## 废弃流程

技能不再维护时，不应静默消失——已安装该技能的代理会因找不到匹配而失败。

### 废弃三阶段

| 阶段 | 操作 | 用户可见状态 |
|------|------|-------------|
| **标记废弃** | SKILL.md 顶部加废弃标注 + 替代方案链接 | 技能仍可用，但推荐迁移 |
| **停止更新** | 不再接受 PR、不更新 digest | 功能性冻结 |
| **归档** | 从 `.well-known/` 移除，仓库标记为 archived | 不再被发现 |

### 废弃标注格式

```markdown
---
name: old-skill
description: （原描述）
deprecated: true    # ⚗️ 提案中——见 R13
---

> **废弃**：此技能自 v2.0 起不再维护。替代方案：[new-skill](../new-skill/SKILL.md)。
> 在 description 中补"不再维护"可减少误触发：
> `description: （原描述）不再维护——见 [new-skill]`
```

### 归档前检查清单

- [ ] 主 SKILL.md 顶部标注废弃 + 替代方案
- [ ] description 已反映废弃状态（含"不再维护/已废弃"关键词）
- [ ] 从 `.well-known/agent-skills/index.json` 移除条目
- [ ] 替代技能已准备好并可被发现
- [ ] 仓库或其目录标记为 `archived`

### 不推荐的做法

| 做法 | 问题 |
|------|------|
| 直接删除 SKILL.md | 已下载技能的代理引用断裂 |
| 保留但不标注废弃 | 代理继续匹配并加载过时指令 |
| 只更新 description 不更新 body | 声明-行为不一致（AST10 风险）|

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R13] | <https://github.com/agentskills/agentskills/issues/90> | Proposal: Skill Relationship Fields | `version`、`deprecated`、`prerequisite-skills`、`related-skills` 字段提案 |

> 完整引用见主 SKILL.md 参考文献表。
