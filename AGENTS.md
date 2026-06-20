# Agent Prompt Skills — Agent Guide

## 命令

```bash
# 检查 skills/ 下所有 markdown 文件
markdownlint --config .markdownlint.toml skills/*/SKILL.md

# 自动修复可修复问题
markdownlint --fix --config .markdownlint.toml skills/*/SKILL.md

# 编辑 SKILL.md 后同步更新 .well-known/agent-skills/index.json 中的 digest
# 先运行 sha256sum 获取新值，再更新 index.json 中的 "digest" 字段
sha256sum skills/writing-agent-docs/SKILL.md skills/structuring-agents-md/SKILL.md skills/writing-skills/SKILL.md
```

## 边界

| 层级 | 规则 |
|------|------|
| **Never** | 勿启用 `.markdownlint.toml` 中禁用的规则（注释已说明原因） |
| **Always** | 修改 `.md` 后运行 `markdownlint` 验证 |
| **Always** | 修改 `skills/*/SKILL.md` 后，同步更新 `.well-known/agent-skills/index.json` 中对应 `digest` 字段 |
| **Always** | 新增/移除技能目录时同步更新 `.claude-plugin/plugin.json` 和 `.well-known/agent-skills/index.json` |
| **Note** | 通过 raw URL 使用 `npx skills add` 时，仓库根目录必须配置 `.well-known/agent-skills/index.json`，否则无法发现技能 |
| **Ask** | 需修改 `.markdownlint.toml` 配置时先确认 |
| **Always** | 本仓库是元技能仓库——修改任何技能前，先加载该技能本身及其依赖链（`writing-agent-docs` → `writing-skills` / `structuring-agents-md`） |

## 层级与作用域

本仓库使用 AGENTS.md 层级组织开发指南。每个 `skills/<skill>/` 目录下的 AGENTS.md 仅管辖该技能目录范围，根 AGENTS.md 覆盖全局。

目前无子目录级 AGENTS.md，如有需要按以下规则添加：

- 仅记录该技能目录特有内容（测试方法、特殊约定）
- 不重复根文件已声明的内容
- 与根文件冲突时以子目录为准

## 提交格式

Conventional Commits。title 英文，body 中文（可选）。

```
feat: add structured-agents-md skill
fix: correct description in writing-agent-docs
docs: update README with skill dependency table
chore: update .well-known digest for writing-skills
```

## 写作原则

本仓库服务于编写代理文档本身，修改技能内容时应遵循技能自身所教授的方法：

1. **确定性约束优先**——markdownlint 能强制的格式问题不写入文档规则
2. **增量迭代**——不为假设场景写规则，只针对观察到的真实失败
3. **每 token 须自证价值**——简洁优先，不重复代理已能自行发现的内容
4. **示例优先于解释**——正反示例胜过三段文字描述
5. **肯定指令优先**——否定指令必须配明确的肯定替代

## 技能依赖链

修改一个技能前，确保相关依赖技能已加载：

```
writing-agent-docs（基础）
├── writing-skills → 引用 test-driven-development
└── structuring-agents-md
```

例如：修改 `writing-skills/SKILL.md` 时，应加载 `writing-agent-docs` + `writing-skills` 自身 + `test-driven-development`。

## 内容规则

- `skills/writing-skills/examples/`、`skills/structuring-agents-md/reference/` 等子文档只修改格式问题，不修改实质性内容；`SKILL.md` 索引文件允许结构编辑和措辞优化
- 文件名保持英文连字符风格，与现有命名一致
- 参考文件（如 `reference/*.md`、`examples/*.md`）必须在开头标注来源
- 修改后运行 `markdownlint` 验证格式
