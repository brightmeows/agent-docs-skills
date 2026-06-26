# 自动生成技能审查指南

> 人类作者参考：代理（如 Hermes Agent `/learn`、OpenClaw 等）可自动创建 SKILL.md。
> 自动生成的技能需要人工审查才能确保质量、安全和可遵从性。

---

## 背景

2026 年中，多个工具开始支持自动技能生成：

- **Hermes Agent `/learn`**（Nous Research, 2026-06-24 [R1]）：从目录、URL、对话或笔记自动生成 SKILL.md
- **Hermes Agent `skill_manage`（auto）**：复杂任务完成后自动保存技能作为程序化记忆
- **OpenClaw / 其他代理**：通过 `/learn` 风格命令将工作流转化为技能

自动生成降低了创建技能的门槛，但引入了新的质量控制问题。

---

## 自动生成技能的常见质量问题

### 1. Description 总结了工作流（最常见）

自动生成的技能最容易犯的错误——description 描述了过程而非功能和触发条件。

```yaml
# 坏（自动生成常见）：总结了工作流
description: 运行代码审查，检查代码质量，提交反馈给开发者
# 好：功能 + 触发条件
description: 对拉取请求进行自动化代码审查和反馈。在提交 PR 审查或需要质量评估时使用。
```

### 2. Body 过于冗长

自动生成倾向于堆砌所有已知信息，而非精心选择最小高信号 token 集。

```markdown
# 坏：自动生成的冗余内容
## 第一步：准备
确保你已经安装了 Git（如果还没安装，去 https://git-scm.com 下载……）

# 好：简洁
## 前置条件
Git 已安装并配置
```

### 3. 代码示例使用占位符

自动生成不知道项目的真实数据，倾向使用 `"string"`、`"YOUR_VALUE"` 等占位符。

```python
# 坏：自动生成的占位符
client = Client(api_key="YOUR_API_KEY")  # ← 代理会字面复制

# 好：真实值或环境变量指引
client = Client(api_key=os.environ["MY_API_KEY"])
```

### 4. 缺少常见错误和边缘情况

自动生成通常只覆盖快乐路径（happy path），忽略错误恢复和阻塞处理。

### 5. 工作流脱离实际

自动生成的命令和步骤可能依赖不存在的工具或过时的 API。

### 6. 安全考虑缺失

自动生成的技能可能包含不安全的默认配置、过度授权的 `allowed-tools`、或引用未审计的外部资源。

---

## 审查清单

### 内容审查

- [ ] description 是“做什么 + 何时使用”格式，**未总结工作流**
- [ ] description 包含具体触发条件和搜索关键词
- [ ] body <500 行，重内容已拆分到 `references/`
- [ ] 没有代理已预训练的内容（标准语法、常见 API）
- [ ] 没有推测性规则（仅针对观察到的真实失败）
- [ ] 否定指令配了肯定替代（或确属硬性禁令）
- [ ] 代码示例使用真实值，不用占位符
- [ ] 示例完整可运行、来自真实场景
- [ ] 包含常见错误和边缘情况章节
- [ ] 命令带精确 flag，路径用正斜杠

### 安全审查

- [ ] 无硬编码凭证或密钥
- [ ] `allowed-tools` 未过度授权
- [ ] 脚本文件安全（不执行危险操作）
- [ ] description 不暴露敏感信息
- [ ] description 与实际行为一致——body 不执行 description 未声明的操作
- [ ] 代码示例与配置模板已审阅（无来源不明的可执行片段）
- [ ] 技能若写代理记忆文件（`SOUL.md`/`MEMORY.md` 等），写入内容已审阅
- [ ] 外部 URL 引用为静态内容（无可变外部链接——[security.md](../references/security.md) 详述）

### 结构审查

- [ ] `name` 与父目录名一致，仅小写字母/数字/连字符
- [ ] YAML frontmatter 格式正确（`name` + `description` 必需）
- [ ] 引用保持一层深度（无深层嵌套）
- [ ] 重内容下沉到 `references/` 或 `scripts/`
- [ ] `authoring/` 目录文件仅人类作者需要，代理常规任务不加载

### 实用性审查

- [ ] 走查：代理能否找到（CSO）、能否理解（结构）、能否遵从（清晰）
- [ ] 所有路径与命令真实存在
- [ ] 关键操作有验证 / 确认步骤
- [ ] 工作流有显式完成标准和阻塞升级路径

---

## 自动生成 vs 手写 场景选择

| 场景 | 自动生成 | 手写 |
|------|---------|------|
| 从现有文档/API 参考快速创建技能 | ✅ `/learn` 最合适 | ❌ 编码速度慢 |
| 捕获调试/部署工作流的程序化记忆 | ✅ 代理自行保存 | ❌ 人类容易遗漏细节 |
| 需要精确控制措辞和触发条件 | ❌ 需大幅修改 | ✅ 从头把控 |
| 纪律执行型（行为安全关键） | ❌ 自动生成质量不可控 | ✅ 必须人工精修 |
| 技能用于生产环境/团队分发 | ⚠️ 生成后必须按审查清单逐项过 | ✅ 推荐 |
| 个人实验性技能 | ✅ 快速生成即用 | ⚠️ 视需要 |

---

## 自动生成后的人工优化路径

1. **运行审查清单**——逐项检查
2. **重写 description**——最常出问题，确保“做什么 + 何时使用，不总结工作流”
3. **精简 body**——删除冗余、合并重复、拆分 >500 行的内容
4. **替换占位符**——用项目真实数据替代所有 `"string"`、`"YOUR_VALUE"` 等
5. **补充边缘情况**——添加常见错误、阻塞升级路径、完成标准
6. **安全检查**——检查凭证、`allowed-tools`、外部引用
7. **验证**——pre-commit / `skills-ref validate` 通过
8. **走查**——模拟代理发现和加载场景

---

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|---------|
| [R1] | [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) | Hermes Agent Skills System | 开源 self-improving agent；`/learn` 从目录/URL/对话自动创建 SKILL.md；三级渐进披露 |

## 本地参考

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [writing-skill-md SKILL.md](../SKILL.md) | 主技能文件，包含技能创建清单和自检项 |
| [L2] | [security.md](../references/security.md) | 安全考虑的完整参考 |
| [L3] | [claude-search-optimization.md](../references/claude-search-optimization.md) | CSO 完整规则和 description 编写规范 |

[R1]: https://hermes-agent.nousresearch.com/docs/user-guide/features/skills
