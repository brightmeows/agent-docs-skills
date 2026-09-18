# Agent Docs Skills — Agent Guide

## 仓库定位

本仓库是**指导代理生成代理用文档的元技能集合**。

- **使用者**：AI 编码助手（OpenCode、Claude Code 等）
- **产出物**：AGENTS.md、SKILL.md、.cursor/rules、系统提示词等 agent-facing 文档
- **核心目标**：让代理能独立生成高质量、可发现、可遵从的代理文档

因此本仓库技能的**主流程面向代理可执行的操作**。需要人类深度参与的方法论（如完整 TDD subagent 压力测试）作为**人类作者参考**保留，不作为代理执行的主路径——见 [writing-skill-md/authoring/tdd-validation.md](skills/agent-docs-skills/writing-skill-md/authoring/tdd-validation.md)。

## 工具无关原则

本仓库的通用指导**工具无关**——适用于 OpenCode、Claude Code、Cursor、Gemini CLI、Copilot 等任意代理。写内容时区分：

- **跨工具通用**（AGENTS.md 标准、SKILL.md 开放标准、写作原则、实证数据）→ 默认，无需标注
- **特定工具专属**（Claude Code hooks/subagents、Cursor `.mdc`、OpenCode `opencode.json` 等）→ **必须标注工具名**，并说明其它工具的等价或不等价

不把单一工具的机制写成通用做法。

## 技能依赖链

技能间的依赖关系（修改前的读取义务见下方 Always 边界）：

```
writing-agent-docs（基础原则 + 各代理文档共通部分）
├── writing-skill-md（SKILL.md 格式——跨项目级/个人级）
├── structuring-project-agent-md（项目级配置：AGENTS.md / CLAUDE.md / .cursor/rules）
└── structuring-personal-agent-md（个人级配置：CLAUDE_GLOBAL.md / 个人 AGENTS.md）
```

**引用原则**：依赖方向决定引用方向——上游技能不引用下游技能（无反向引用）。

## 命令

```bash
# pre-commit（6 个任务：markdownlint + well-known list + plugin skills list + digest 检查 + name 一致性 + 格式检查）
pre-commit run --all-files

# 提交时自动触发钩子，也可手动指定单个任务
pre-commit run check-well-known-digest       # index.json digest 与 SKILL.md 匹配
pre-commit run check-skill-name-consistency  # frontmatter name/description 与 index.json 一致
pre-commit run check-plugin-skills-list      # marketplace.json skills 与技能目录一致
pre-commit run check-skill-md-format         # frontmatter 字段格式与 body 行数合规

# 编辑 SKILL.md 后更新 digest（hook 自动校验，此为手动更新命令）
sha256sum skills/agent-docs-skills/*/SKILL.md
```

## 提交格式

Conventional Commits，全程中文。

```
feat: 新增结构化代理文档技能
docs: 更新 writing-skill-md 的目录结构说明
chore: 更新 writing-skill-md 的 well-known digest
```

## 边界

### Always

- 修改 `.md` 后通过 `pre-commit run markdownlint` 验证（pre-commit 中以 `--config .markdownlint.toml` 覆盖默认规则，勿直接调用 markdownlint-cli2）
- 修改 `skills/agent-docs-skills/*/SKILL.md` 后，同步更新 `.well-known/agent-skills/index.json` 中对应 `digest` 字段（由 `check-well-known-digest` hook 强制）
- 新增/移除技能目录时同步更新 `.well-known/agent-skills/index.json` 与 `.claude-plugin/marketplace.json` 的 `skills` 数组（后者为 `npx skills` 提供分组显示，缺则技能平铺无组名）
- **修改任何技能前，必须先完整读取本仓库内的所有前置技能文档。** 依赖关系见[技能依赖链](#技能依赖链)——本仓库是元技能仓库，技能本身即是规范

### Ask

- 需修改 `.markdownlint.toml` 配置时先确认

### Never

- 勿启用 `.markdownlint.toml` 中禁用的规则（注释已说明原因）

### Note

- 通过 raw URL 使用 `npx skills add` 时，仓库根目录必须配置 `.well-known/agent-skills/index.json`，否则无法发现技能

## 内容规则

- 修改后通过 `pre-commit run markdownlint` 验证格式
- **引用规范**：
  - 外部来源（论文、官方文档、博客）使用 `[R1]`、`[R2]`… 格式，在文末 `## 参考文献` 表格中列明编号、URL、标题、核心内容；**URL 列使用 `<url>` 裸链接格式**而非 `[text](url)`，避免与标题列重复
  - 本地文件（`references/`、`scripts/`、`authoring/` 等）使用 `[L1]`、`[L2]`… 格式，在文末 `## 本地参考` 表格中列明编号、文件路径、用途
  - 正文中仅标注引用编号，不附带作者、机构、年份等元信息——完整信息在对应表格中
  - 仅在正文使用 `[描述文字][Rx]` 时才需在文件末尾保留 `[Rx]: url` 定义；仅使用 `（[Rx]）` 或 `[Rx]` 时不需要
- **避免易过期信息**（降低维护频率）：
  - **先问能否升维**：能表述为判断标准（“什么是对的”）的，不表述为当前状态（“世界是什么样的”）——判断标准不过时。详见 writing-agent-docs M.1
  - **枚举→分类**：逐个列举个体改为按类型/类别分组（类型稳定，个体多变）。如工具配置文件表合并同类、市场平台按类型而非名称列举
  - **数字→定性/引用编号**：正文用定性描述（“大规模”“高比例”“快速增长”），精确数字放参考文献表“核心内容”列，正文仅标 `[Rx]`；引用源更新时正文不动，只改参考表一行
  - **版本/日期下沉**：正文去掉版本号与日期（用“较新版本”“近期”“已发布”“草案阶段”等状态词），版本与日期放参考文献或官方文档链接
  - **例外**：安全警示等需精确度论证紧迫性的场景，可保留精确数字，但同样建议正文定性 + 参考表精确

## 维护指南

### 新增内容归属判断

新内容需要写入本仓库时，按以下路径决策：

```
有新内容要添加
├── 属于现有技能范围？
│   ├── 是 → 进入该技能目录
│   └── 否 → 属于哪个技能？（见[技能依赖链](#技能依赖链)）
│       ├── 基础原则 + 各代理文档共通部分 → writing-agent-docs
│       ├── SKILL.md 格式 → writing-skill-md
│       ├── 项目级配置 → structuring-project-agent-md
│       └── 个人级配置 → structuring-personal-agent-md
└── 不属于任何现有技能？
    └── 确认是否需要新增技能目录（见 writing-skill-md“何时创建技能”）
```

### 内容放置决策

确定归属技能后，按以下规则决定放在文件系统的哪个位置：

| 内容类型 | 放哪 | 触发条件 |
|---------|------|---------|
| **原则、概念、简短模式**（≤50 行）| 内联在 SKILL.md 正文 | 内容精简，无需跳转即可理解 |
| **重量级参考**（>50 行）| 下沉到 `references/` | 内容篇幅长，代理按需加载 |
| **可复用脚本/工具** | 下沉到 `scripts/` | 内容是可执行代码 |
| **静态模板/资源** | 下沉到 `assets/` | 内容用于输出嵌入 |
| **人类作者参考** | 下沉到 `authoring/` | 代理常规任务不加载 |

> **参照写作-skill-md 的目录结构规范**：`references/` 按主题分子目录，不按来源分（自撰 vs 转载）。

### 归属优先顺序

内容先检查能否**就地归入当前文件的相关章节**（非索引类），就近归并、免一次加载跳转；同文件无合适归处时才按上表下沉到子文件。

### 引用文件同步

新增或修改参考文件后：

1. 如果参考文件被 SKILL.md 引用 → 更新 SKILL.md 的 `## 本地参考` 表，添加/修改对应 `[Lx]` 条目
2. 如果修改了 SKILL.md → 重新计算 digest（`sha256sum`），更新 `.well-known/agent-skills/index.json`
3. 如果参考文件引用了其他文件 → 确保引用路径正确（使用相对路径，遵循依赖方向）
4. 如果新增/删除了参考文件 → 运行 `pre-commit run --all-files` 检查一致性
