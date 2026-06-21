# 实证数据参考

> structuring-project-agent-md 参考文件：AGENTS.md 实证数据汇总。主文中的具体数字指向本文件。

## 采用情况

AGENTS.md 已被 **60,000+ 开源仓库**采用、被 30+ 工具原生支持（Codex、Copilot、Cursor、Windsurf、Gemini CLI、Devin、Amp、Claude Code 等）。

GitHub 对 2,500+ 仓库的实证分析显示测试指令在 75% 的高质量 AGENTS.md 中出现——频率最高（Nigh, 2025）。

## Lulla et al. (2026) —— 运行效率

Lulla, Mohsenimofidi, Galster, Zhang, Baltes, Treude（Singapore Management Univ. / Heidelberg / Bamberg / King's College London）。ICSE JAWs 2026。

在 10 个仓库、124 个 PR 中测量（OpenAI Codex / gpt-5.2-codex，配对实验）：

- 有 AGENTS.md 的任务**中位运行时间减少 28.6%**（均值 20.3%）
- **中位输出 token 减少 16.6%**（均值 20.1%）
- 任务完成行为可比——效率提升不以质量为代价

## Gloaguen et al. (2026) —— 成功率与行为（ETH Zurich）

Gloaguen, Mündler, Müller, Raychev, Vechev（ETH Zurich SRI Lab）。ICLR 2026 Workshop。
论文：[arXiv:2602.11988](https://arxiv.org/abs/2602.11988)——《Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?》。

在 12 个仓库、138 个任务实例（AGENTbench）+ SWE-bench Lite 上测量（4 个 agent × 多模型）：

- LLM 自动生成的 context file **一致降低成功率**（均值 −0.5% ~ −2%），**推理成本增加 20%+**
- 手写 context file 仅带来 **+4% 边际提升**——且仅限极简精确的文件
- 自动生成使 agent **多走 2.45–3.92 步/任务**（重复 agent 本可自行发现的内容）
- **冗余内容使推理成本增加 23%、成功率下降 2%**；不必要指令使推理 token 增加 14-22%
- **Context Map 价值有限**：目录映射对实现任务的文件发现加速不显著——代理已能自主导航。真实价值在于新会话的架构定向（spec 编写、错误分类、ADR 撰写），而非实现代理的导航捷径
- 移除现有文档后 LLM 生成文件反而 +2.7%——说明生成内容多与现有文档冗余

## 内容分类实证

- **Chatlatanagulchai et al. (2025)** "Agent READMEs"：context file 集中于功能性指令（构建/测试/实现），非功能性关注（性能/安全）稀少
- **Mohsenimofidi et al. (2026)** MSR 2026：最常见类别为 conventions、architecture、project description——即代理最难自行推断的 core 项目知识

## 标准化趋势

**NIST AI Agent Standards Initiative（2026-02）**——可互操作、安全的 AI agent 标准框架，为 agent 配置文件标准化方向提供背书。

## OpenAI Codex 默认截断

32 KiB，超出部分静默丢弃。
