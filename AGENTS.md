# Agent Docs Skills — Agent Guide

## 仓库定位

本仓库是**指导代理生成代理用文档的元技能集合**。

- **使用者**：AI 编码助手（OpenCode、Claude Code 等）
- **产出物**：AGENTS.md、SKILL.md、.cursor/rules、系统提示词等 agent-facing 文档
- **核心目标**：让代理能独立生成高质量、可发现、可遵从的代理文档

因此本仓库技能的**主流程面向代理可执行的操作**。需要人类深度参与的方法论（如完整 TDD subagent 压力测试）作为**人类作者参考**保留，不作为代理执行的主路径——见 [writing-skill-md/authoring/tdd-validation.md](skills/writing-skill-md/authoring/tdd-validation.md)。

## 命令

```bash
# pre-commit（4 个任务：markdownlint + list 检查 + digest 检查 + spec 检查）
pre-commit run --all-files

# 提交时自动触发钩子，也可手动指定单个任务
pre-commit run check-well-known-digest       # index.json digest 与 SKILL.md 匹配
pre-commit run check-skill-md-spec           # frontmatter name/description 与 body 行数合规

# 编辑 SKILL.md 后更新 index.json 中的 digest
sha256sum skills/*/SKILL.md
```

## 边界

### Always

- 修改 `.md` 后通过 `pre-commit run markdownlint` 验证（pre-commit 中以 `--config .markdownlint.toml` 覆盖默认规则，勿直接调用 markdownlint-cli2）
- 修改 `skills/*/SKILL.md` 后，同步更新 `.well-known/agent-skills/index.json` 中对应 `digest` 字段
- 新增/移除技能目录时同步更新 `.well-known/agent-skills/index.json`（`.claude-plugin/plugin.json` 依赖默认 `skills/` 目录扫描，无需维护技能列表）
- 发布新版本（release/tag）时，同步更新 `.claude-plugin/plugin.json` 的 `version` 字段和 `README.md` 中的安装命令版本引用
- **修改任何技能前**，必须先读取 `writing-agent-docs`（基础写作原则）和对应的领域 skill（`writing-skill-md`、`structuring-project-agent-md` 或 `structuring-personal-agent-md`），并按其要求执行——本仓库是元技能仓库，技能本身即是规范

### Ask

- 需修改 `.markdownlint.toml` 配置时先确认

### Never

- 勿启用 `.markdownlint.toml` 中禁用的规则（注释已说明原因）

### Note

- 通过 raw URL 使用 `npx skills add` 时，仓库根目录必须配置 `.well-known/agent-skills/index.json`，否则无法发现技能

## 提交格式

Conventional Commits。title 英文，body 中文（可选）。

```
feat: add structured-agents-md skill
chore: update .well-known digest for writing-skill-md
```

## 技能依赖链

技能间的依赖关系（修改前的读取义务见上方 Always 边界）：

```
writing-agent-docs（基础写作原则）
├── writing-skill-md（SKILL.md 格式——跨项目级/个人级）
├── structuring-project-agent-md（项目级配置：AGENTS.md / CLAUDE.md / .cursor/rules）
└── structuring-personal-agent-md（个人级配置：CLAUDE_GLOBAL.md / 个人 AGENTS.md）
```

## 内容规则

- `skills/writing-skill-md/authoring/`、`skills/structuring-project-agent-md/reference/` 等子文档只修改格式问题，不修改实质性内容；`SKILL.md` 索引文件允许结构编辑和措辞优化
- 文件名保持英文连字符风格，与现有命名一致
- 参考文件（如 `reference/*.md`、`authoring/*.md`）必须在开头标注来源
- 修改后通过 `pre-commit run markdownlint` 验证格式
