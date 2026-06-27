# README vs AGENTS.md vs CONTRIBUTING.md 职责分离

> `structuring-project-agent-md` 参考文件。
> 项目仓库中常出现三个面向不同受众的顶层 Markdown 文件，
> 职责混淆会导致内容重复、维护漂移和代理解析歧义。

---

## 三文件职责

| 文件 | 受众 | 管什么 | 不管什么 |
|------|------|--------|---------|
| `README.md` | **人类开发者** | 项目做什么、安装方式、快速开始、使用示例 | 代理指令、构建命令细节、边界规则 |
| `AGENTS.md` | **AI 编码代理** | 构建/测试命令、代码约定、Always/Ask/Never 边界 | 用户安装指南、API 参考、贡献流程 |
| `CONTRIBUTING.md` | **人类贡献者** | 如何提交 PR、代码审查标准、分支策略 | 代理指令、运行时命令 |

## 典型混淆场景

| 混淆 | 正确做法 |
|------|---------|
| README 中写了 `pnpm test` 等代理指令 | 移入 AGENTS.md，README 仅保留“见 AGENTS.md”一行 |
| AGENTS.md 中写了“如何安装项目依赖” | 移入 README——代理可从 README 自行读取 |
| CONTRIBUTING.md 与 AGENTS.md 的测试命令不一致 | AGENTS.md 维护真理源，CONTRIBUTING.md 引用 AGENTS.md |
| README 中含“不需要 sudo”等安全边界 | 移入 AGENTS.md Never 区块 |

## 协同原则

### 1. AGENTS.md 是代理的真理源

代理指令应统一放在 AGENTS.md 中，README 和 CONTRIBUTING.md 不重复。

```markdown
# README.md——好做法：指向 AGENTS.md
## 与 AI 编码助手配合
本项目提供 AGENTS.md 帮助 AI 编码助手理解项目约定。
详见：[AGENTS.md](AGENTS.md)

# README.md——坏做法：代理指令散落
## 开发
用 pnpm dev 启动（本命令也在 AGENTS.md 中定义了）
用 pnpm test 跑测试
```

### 2. 内容重合时归 AGENTS.md

同样的命令或规则在 README 和 AGENTS.md 中都出现时，以 AGENTS.md 为代理的真理源：
README 中只需一行引用，无需完整复制。

### 3. CONTRIBUTING.md 面向人，不写代理指令

```markdown
# CONTRIBUTING.md——好
## 提交 PR
1. 确保测试通过（`pnpm test`）
2. 确保 lint 通过（`pnpm lint`）
3. 提交 PR 并添加 `review` 标签
# 注：上述命令在 AGENTS.md 中有等效的精确版本

# CONTRIBUTING.md——坏
## 提交 PR
1. 运行 pnpm test -- --run --reporter dot  # 精确 flag 应只在 AGENTS.md
```

## 验证方法

| 检查 | 方法 |
|------|------|
| 三文件无重复构建/测试命令 | `grep -n 'pnpm\|npm run\|cargo' README.md AGENTS.md CONTRIBUTING.md` |
| README 不写代理边界规则 | 搜索 `Never\|Ask First\|Always`——应在 AGENTS.md 中 |
| AGENTS.md 不写安装指南 | 搜索 `install\|getting started\|quick start`——应非代理内容 |

---

## 本地参考

| 编号 | 文件 | 用途 |
|------|------|------|
| [L1] | [content-decisions.md](./content-decisions.md) | 内容决策 5 维度评分——README vs AGENTS.md 边界判断依据 |
