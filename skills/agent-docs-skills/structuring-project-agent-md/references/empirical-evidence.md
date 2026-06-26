# 实证数据参考

> structuring-project-agent-md 参考文件：AGENTS.md 实证数据汇总。主文中的具体数字指向本文件。

## 采用情况

AGENTS.md 已被 **60,000+ 开源仓库**采用、被 30+ 工具原生支持（Codex、Copilot、Cursor、Windsurf、Gemini CLI、Devin、Amp、Claude Code 等）。

GitHub 对 2,500+ 仓库的归纳分析将 testing 列为高质量 AGENTS.md 的六个核心领域之一（[R8]）。

## Lulla et al. (2026) —— 运行效率

论文：[R1]——《On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents》。

在 10 个仓库、124 个 PR 中测量（OpenAI Codex / gpt-5.2-codex，配对实验）：

- 有 AGENTS.md 的任务**中位运行时间减少 28.6%**（均值 20.3%）
- **中位输出 token 减少 16.6%**（均值 20.1%）
- 任务完成行为可比——效率提升不以质量为代价

## Gloaguen et al. (2026) —— 成功率与行为

论文：[R2]——《Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?》。

在 12 个仓库、138 个任务实例（AGENTbench）+ SWE-bench Lite 上测量（4 个 agent × 多模型）：

- LLM 自动生成的 context file **一致降低成功率**（均值 −0.5% ~ −2%），**推理成本增加 20%+**
- 手写 context file 仅带来 **+4% 边际提升**——且仅限极简精确的文件
- 自动生成使 agent **多走 2.45–3.92 步/任务**（重复 agent 本可自行发现的内容）
- **冗余内容使推理成本增加 23%、成功率下降 2%**；不必要指令使推理 token 增加 14-22%
- **Context Map 价值有限**：目录映射对实现任务的文件发现加速不显著——代理已能自主导航。真实价值在于新会话的架构定向（spec 编写、错误分类、ADR 撰写），而非实现代理的导航捷径
- 移除现有文档后 LLM 生成文件反而 +2.7%——说明生成内容多与现有文档冗余

**原因**：代理忠实跟随生成指令，但生成内容含微妙不准确，导致探索范围扩大、推理成本上升。产业实测印证（[R7]）：最好的 AGENTS.md 带来相当于 Haiku→Opus 的质量跃升，最差的比没有 AGENTS.md 更糟。

**关键区分——问题在“自动生成”，不在文件本身**：同期 Lulla et al.（上方）测得手写 AGENTS.md 使运行时间 −28.6%、token −16.6%。即手写精简提升效率，自动生成损害效率。`/init` 等结果须手工重写，而非弃用 AGENTS.md。

## McMillan (2026) —— 结构变量与即时遵从

论文：[R3]——《Instruction Adherence in Coding Agent Configuration Files: A Factorial Study of Four File-Structure Variables》。

在 1,650 个 Claude Code CLI 会话（16,050 函数级观测，Sonnet 4.6 主力 + Opus 4.6 交叉验证）上做因子实验，检验四个文件结构变量：

- **文件大小、指令位置、文件架构、相邻文件冲突**——四个变量均未产生可检测的对比效应（多重检验校正后）
- size 与 conflict 的 null 有贝叶斯因子支持（BF10 0.05–0.10）；position 与 architecture 为未能拒绝但无 BF 支持
- **最大效应是会话内**：每多生成一个函数，单步遵从概率约 −5.6%（OR=0.944），关系非单调
- 测试对象为单条 trivial 标注指令的**即时遵从**，非整体成功率或成本——故与 Gloaguen（成本）/ dos Santos（相关性）测的维度不同，三者互补而非矛盾

**启示**：不必过度优化指令的具体位置与文件架构；真正该关注的是会话长度管理（任务分解、定期重置上下文）。

## 内容分类实证

- **Chatlatanagulchai et al. (2025)** “Agent READMEs”（[R4]）：context file 集中于功能性指令（构建/测试/实现），非功能性关注（性能/安全）稀少
- **Mohsenimofidi et al. (2026)** MSR 2026（[R5]）：最常见类别为 conventions、architecture、project description——即代理最难自行推断的 core 项目知识

## 标准化趋势

**[R6]（2026-02）**——可互操作、安全的 AI agent 标准框架，为 agent 配置文件标准化方向提供背书。

## OpenAI Codex 默认截断

32 KiB，超出部分静默丢弃。

## 参考文献

| 编号 | 链接 | 标题 | 核心内容 |
|------|------|------|----------|
| [R1] | <https://arxiv.org/abs/2601.20404> | On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents | 手写 AGENTS.md 使运行时间减少 28.6%，输出 token 减少 16.6% |
| [R2] | <https://arxiv.org/abs/2602.11988> | Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents? | 自动生成 context file 降低成功率并增加推理成本，手写精简文件仅有 +4% 边际提升 |
| [R3] | <https://arxiv.org/abs/2605.10039> | Instruction Adherence in Coding Agent Configuration Files: A Factorial Study of Four File-Structure Variables | 文件结构变量（大小/位置/架构/冲突）对即时遵从均无显著效应 |
| [R4] | <https://arxiv.org/abs/2511.12884> | Agent READMEs | context file 集中于功能性指令（构建/测试/实现），非功能性关注稀少 |
| [R5] | <https://arxiv.org/abs/2510.21413> | MSR 2026 论文：An Empirical Study of Context Files for AI Coding Agents | 最常见类别为 conventions、architecture、project description |
| [R6] | <https://www.nist.gov/artificial-intelligence/ai-agent-standards-initiative> | NIST AI Agent Standards Initiative | AI agent 标准框架，为 agent 配置文件标准化方向提供背书 |
| [R7] | <https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files> | How to Write Good AGENTS.md Files | 产业实测：最佳 AGENTS.md 相当于 Haiku→Opus 质量跃升，最差比没有更糟 |
| [R8] | <https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/> | How to Write a Great Agents.md: Lessons from Over 2,500 Repositories | GitHub 分析 2,500+ 仓库，testing 列为高质量 AGENTS.md 六个核心领域之一 |
