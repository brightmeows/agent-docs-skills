# 作用域与加载顺序

> `writing-agent-docs` 参考文件。
> 面向代理的文档按存放位置分为两个作用域；不同文件的加载时机和优先级决定你应该写到哪里。

---

## 作用域总览

### 项目级（Project-level）

位于仓库内的配置文件，告诉代理**在此项目中如何工作**。

**文件清单**

| 文件 / 目录 | 用途 | 工具 |
|-------------|------|------|
| `AGENTS.md` | 跨工具标准，项目约定 / 命令 / 边界 | 25+ 工具原生支持 |
| `CLAUDE.md` | 项目级 Claude Code 配置文件 | Claude Code |
| `.cursor/rules/*.mdc` | 文件模式匹配规则，按 glob 注入 | Cursor、OpenCode 等 |
| `GEMINI.md` | 项目级 Gemini CLI 配置 | Gemini CLI |
| `.junie/guidelines.md` | 项目级 Junie 配置 | JetBrains Junie |
| `.claude/` | Claude Code 项目级指令目录（含 Commands）| Claude Code |

**写作目标**：精炼——只包含该项目特有的、代理无法从代码自行推断的内容。工具链能强制的约束指向工具而非写入文档。

**维护者**：项目团队，经代码审查，随项目演变。

### 个人级（Personal-level）

位于个人目录中的配置文件，告诉 agent **该如何为你工作**。

**文件清单**

| 路径 | 用途 | 工具 |
|------|------|------|
| `~/.claude/CLAUDE_GLOBAL.md` | 全局行为指令，优先于项目 CLAUDE.md | Claude Code |
| `~/.agents/AGENTS.md` | 个人级 AGENTS.md（部分工具支持）| 通用 |
| `~/.cursor/rules/`（全局）| 全局 Cursor 规则 | Cursor |
| `~/.config/opencode/opencode.json` | 全局 OpenCode 工具配置 | OpenCode |
| `~/.claude/settings.json` | Claude Code 全局设置 | Claude Code |
| `~/.claude/skills/` / `~/.agents/skills/` | 个人技能（跨项目可用）| Claude Code / Codex 等 |
| `~/.cursor/settings.json` | Cursor 全局设置 | Cursor |

**写作目标**：个人偏好、全局规则、通用工作流。不写项目特有内容。保持简短（5–15 行），避免挤压项目级配置的 token 预算。

**维护者**：本人，不进版本控制。

---

## 加载顺序

典型编码代理（Claude Code / OpenCode / Cursor）按以下顺序加载配置：

### 流水线层级

1. **基础模型知识（预训练）**——模型自身的训练数据、通用编程知识
2. **个人级配置**（全局加载一次）
   - `~/.config/opencode/opencode.json`
   - `~/.claude/CLAUDE_GLOBAL.md`
   - `~/.claude/settings.json`、`~/.cursor/settings.json`
   - `~/.claude/skills/*/SKILL.md`、`~/.agents/skills/*/SKILL.md`
3. **项目级配置**（进入仓库时发现）
   - `opencode.json` / `opencode.jsonc`
   - `AGENTS.md`（跨工具标准）
   - `CLAUDE.md`（Claude Code 原生）
   - `.cursor/rules/*.mdc`（Cursor 规则）
   - `GEMINI.md`、`.junie/guidelines.md`、`.claude/`
4. **项目级技能发现**（加载 frontmatter 元数据）
   - `.well-known/agent-skills/index.json`
   - `.claude/skills/*/SKILL.md`、`.cursor/skills/*/SKILL.md`
   - `.agents/skills/*/SKILL.md`、`.opencode/skills/*/SKILL.md`
5. **按需技能加载**——代理遇到特定任务 → 扫描技能描述 → 匹配 → 加载 SKILL.md body

### 优先级规则

**就近优先**：个人级（全局）→ 项目级（仓库根）→ 子目录级别。项目级覆盖个人级冲突内容。

**常驻 vs 按需**：

| 配置类型 | 加载时机 | token 开销 | 适用场景 |
|---------|---------|-----------|---------|
| AGENTS.md | 项目发现时加载（常驻）| 每轮 ~944+ token | 项目上下文、边界、命令 |
| opencode.json | 代理启动时加载（常驻）| 配置项计入上下文 | 工具配置、MCP、权限 |
| .cursor/rules/\*.mdc | 按 glob 匹配注入（常驻）| 匹配时注入 | 文件级规则 |
| SKILL.md | 按需加载 body | 每轮 ~53 token（约 18× 节省）| 任务知识、工作流 |

开发者实测显示，等效内容在 SKILL.md 中每轮约消耗 53 token，而在 AGENTS.md 中常驻条目达 944+ token（约 18 倍）——见 `structuring-project-agent-md/reference/comparison-tools.md`。

**同层优先级**（项目级根目录多文件时）：

| 文件 | 优先级 | 说明 |
|------|--------|------|
| `opencode.json`（项目级）| 最高 | OpenCode 的入口配置 |
| `.cursor/rules/*.mdc` | 按 glob 匹配 | 匹配到即注入，`alwaysApply: true` 强制常驻 |
| `CLAUDE.md` | 高 | Claude Code 原生读取 |
| `AGENTS.md` | 中 | 跨工具 fallback，工具原生格式不存在时读取 |
| `GEMINI.md` | 低 | Gemini CLI |

**技能加载优先级**：代理遇到问题 → 检查技能描述是否匹配 → 匹配后加载 SKILL.md 全文 → 技能 body 中的 **HARD GATE** 是最终指令，不能被项目级或个人级配置覆盖。

### 实战决策

| 场景 | 写在 | 原因 |
|------|------|------|
| 仅影响一个项目 | 项目级 AGENTS.md / CLAUDE.md | 其他项目不受影响 |
| 所有项目中都想要的行为 | 个人级 CLAUDE_GLOBAL.md 等 | 全局生效 |
| 特定任务才加载的知识 | SKILL.md | 按需加载，节省常驻 token |
| 不能被覆盖的强制规则 | SKILL.md HARD GATE | 技能 body 是最终指令 |
| 某个目录下所有文件的编码规则 | `.cursor/rules/*.mdc` + glob | 仅匹配的文件触发注入 |
| 整个项目跨工具共享的行为约定 | AGENTS.md | 跨工具标准，多个工具可读 |

---

## 典型工具的加载差异

### OpenCode

1. 加载 `~/.config/opencode/opencode.json`（个人级）
2. 发现项目根 `opencode.json` / `.opencode/opencode.json`
3. 深合并（project overrides global）
4. 扫描 skills 路径
5. 扫描 references

### Claude Code

1. 加载 `~/.claude/CLAUDE_GLOBAL.md` + `~/.claude/settings.json`
2. 读取项目根 `CLAUDE.md`
3. 读取 `.claude/` 目录中的指令文件
4. 扫描 `.claude/skills/` 技能

### Cursor

1. 加载 `~/.cursor/settings.json`
2. 读取 `.cursor/rules/` 目录下的 `*.mdc` 文件
3. 按 `globs` 匹配注入规则
4. `alwaysApply: true` 的规则始终注入

---

## 参考

- `structuring-project-agent-md/reference/comparison-tools.md` — AGENTS.md vs Skill vs MCP token 对比
- `writing-agent-docs/SKILL.md` — 作用域定义
