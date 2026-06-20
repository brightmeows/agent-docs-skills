# Agent Prompt Skills — Agent Guide

## 命令

```bash
# 检查 skills/ 下所有 markdown 文件
markdownlint --config .markdownlint.toml skills/*/SKILL.md

# 自动修复可修复问题（SKILL.md 仅；子文档格式问题手动处理）
markdownlint --fix --config .markdownlint.toml skills/*/SKILL.md

# 编辑 SKILL.md 后同步更新 .well-known/agent-skills/index.json 中的 digest
# 先运行 sha256sum 获取新值，再更新 index.json 中的 "digest" 字段
sha256sum skills/*/SKILL.md

# pre-commit（3 个并行任务：markdownlint + list 检查 + digest 检查）
pre-commit run --all-files

# 提交时自动触发钩子，也可手动指定单个任务
pre-commit run markdownlint --all-files
pre-commit run check-well-known-list
pre-commit run check-well-known-digest
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
| **Always** | 本仓库是元技能仓库——修改任何技能前，先加载该技能本身及其依赖链 |

## 提交格式

Conventional Commits。title 英文，body 中文（可选）。

```
feat: add structured-agents-md skill
fix: correct description in writing-agent-docs
docs: update README with skill dependency table
chore: update .well-known digest for writing-skills
```

## 技能依赖链

修改前加载依赖（已通过 `writing-agent-docs` 覆盖写作原则，无需重复）：

```
writing-agent-docs（基础）
├── writing-skills → 引用 test-driven-development
├── structuring-agents-md
└── test-driven-development（独立基础技能）
```

## 内容规则

- `skills/writing-skills/examples/`、`skills/structuring-agents-md/reference/` 等子文档只修改格式问题，不修改实质性内容；`SKILL.md` 索引文件允许结构编辑和措辞优化
- 文件名保持英文连字符风格，与现有命名一致
- 参考文件（如 `reference/*.md`、`examples/*.md`）必须在开头标注来源
- 修改后运行 `markdownlint` 验证格式
