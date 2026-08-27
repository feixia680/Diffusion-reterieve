# 摘要（中文翻译）

离散扩散语言模型通过对完整回答进行迭代去噪来并行生成文本。在每个步骤中，模型会对所有被掩码的位置预测暂定 token：高置信预测被提交到输出中，而低置信预测通常被丢弃。本文发现，这些被丢弃的 token 实际上是检索增强生成中非常有价值的“前瞻信号”：即使置信度很低，它们也经常在去噪轨迹早期暴露关键实体，从而使系统能够在最终答案确定之前检索到更强的证据。基于这一观察，作者提出 Self-Augmenting Retrieval for Diffusion Language Models（SARDI），在去噪过程中利用这些 lookahead token 动态构造检索查询。SARDI 无需训练、与具体检索器无关，并可应用于具备推理能力的离散扩散语言模型。在五个多跳问答基准上，SARDI 优于现有训练免的扩散式和自回归式检索基线，同时最高可获得约 8 倍吞吐率。

# 1. TL;DR

这篇论文最重要的不是“又做了一个 RAG 框架”，而是发现了一个 dLLM 独有现象：**low-confidence / discarded token 并不等于无信息 token**。在最终答案形成之前，这些 tentative predictions 已经可能暴露 bridge entity。SARDI 只是这个 observation 的自然结果：把中间态变成检索 query，再让证据反馈给后续 denoising。

# 2. 论文基本信息

- 英文题目：Self-Augmenting Retrieval for Diffusion Language Models
- 中文题目：面向扩散语言模型的自增强检索
- 作者：Paul Jünger, Justin Lovelace, Linxi Zhao, Dongyoung Go, Kilian Q. Weinberger
- 会议：ICML 2026
- 论文：https://arxiv.org/abs/2606.06474
- 方向标签：`Retrieval from Diffusion Dynamics`、`Dynamic RAG`、`Training-free`
- 是否需要训练：否
- 核心任务：multi-hop QA
- 复现定位：本仓库第一优先级 baseline

# 3. 研究背景

传统 RAG 通常在生成前用原问题检索，或在自回归推理过程中根据已经生成的前缀再次检索。dLLM 与 AR LLM 不同：每个 denoising step 会同时给大量 masked positions 产生预测，其中很多预测因为置信度不够最终会被 remask。过去这些低置信预测通常被当成计算副产品直接丢弃。

# 4. Existing Assumption

> 只有最终提交或高置信 token 才包含可靠语义；低置信、最终被 remask 的 token 没有利用价值。

SARDI 的核心贡献就是否定这个假设。

# 5. 核心现象 / Pilot Observation

1. tentative token 在 denoising 早期已经经常出现与最终推理相关的显著实体。
2. 即使某个 token 最终被 remask，它仍可能比 question-only query 更早给出 multi-hop bridge entity。
3. 因而“token 是否最终保留”和“token 是否有检索价值”是两个不同变量。
4. dLLM 的中间态天然带有一种 AR 模型难以获得的 lookahead：模型并未提交答案，但多个未来位置已经产生候选语义。

# 6. 为什么会出现这个现象？

## 作者层面的解释

dLLM 在每个去噪步骤对多个位置并行建模，因此低置信预测虽然不够稳定到可以直接提交，却已经受到问题与当前部分答案的双向语义约束。它们更像“模型正在考虑的候选未来”，而不是随机噪声。

## 我的机制理解

这里应区分两种不确定性：

- **commit uncertainty**：token 是否可靠到可以写入最终答案；
- **retrieval utility uncertainty**：token 是否提供了足够具体的检索线索。

一个实体可以 commit confidence 很低，但 retrieval utility 很高。这正是 SARDI 能工作的根本原因。

# 7. Problem Formulation

把 dLLM 的第 t 个去噪状态记为 `s_t`，其中包含当前已提交 token、mask 位置及各位置 tentative predictions。传统系统只使用问题 `q` 构造检索查询；SARDI 进一步从 `s_t` 中提取 lookahead token，形成动态 query `q_t`，执行：

`state s_t → lookahead tokens → retrieval query q_t → evidence D_t → later denoising`。

研究目标不是单独提高 Recall，而是让提前获得的 evidence 最终提升回答准确率，同时保持 dLLM 的吞吐优势。

# 8. 方法总览

1. 正常执行 dLLM 去噪。
2. 收集中间步骤产生但尚未最终提交的 tentative token。
3. 从中选择有信息的 lookahead 内容，与原问题组合成动态检索 query。
4. 调用任意外部 retriever 获取证据。
5. 把证据反馈给后续 denoising，让模型在答案固定之前得到知识补充。

SARDI 的设计重点是**不训练额外模块**，因此它更像 inference-time orchestration。

# 9. 方法详细解析

SARDI 的关键不是复杂 loss，而是动态利用 denoising state。可把其思想抽象成：

`q_t = f(q, z_t)`，其中 `z_t` 表示当前 tentative predictions；再执行 `D_t = R(q_t)`。

真正值得关注的是 `f`：哪些 token 应进入 query、什么时候 query 已成熟、什么时候早期 hallucinated token 会污染检索。这些问题并未被彻底解决，也正是后续研究空间。

# 10. 算法 / 推理流程

- 输入问题；
- 开始 diffusion denoising；
- 在预定或满足条件的 timestep 读取 tentative predictions；
- 形成动态检索 query；
- 取回 evidence；
- 将 evidence 注入上下文；
- 继续 denoising 到最终答案。

整个过程无需更新 dLLM 或 retriever 参数。

# 11. 实验设置

论文在五个 multi-hop QA benchmark 上比较 training-free diffusion retrieval 与 autoregressive retrieval baseline。核心评价既包括最终 QA 质量，也关注吞吐率，因为 dLLM 的价值之一就是并行生成。SARDI 可配合不同 retriever，强调 retriever-agnostic。

# 12. 主实验结果

- 在五个多跳问答基准上整体优于当前 training-free diffusion / AR retrieval baselines。
- 在保持较强问答质量的同时，报告最高约 `8×` throughput 优势。
- 结果说明中间 tentative token 的收益不是只体现在 retrieval recall，而能传导到最终 answer quality。

# 13. Ablation / Controlled Experiments

这篇论文最值得复现的不是完整主表，而是围绕 query source 做控制：

- question-only retrieval；
- committed-token retrieval；
- tentative-token retrieval；
- 不同 timestep 使用 tentative token；
- 不同 token 过滤策略。

后续实验应进一步加入 entity stability、revision frequency、entropy 等信号，比较谁最能解释 retrieval gain。

# 14. Failure Cases / Case Study

潜在失败来自 hallucinated lookahead：如果一个错误实体在早期恰好很“具体”，它可能触发错误检索，并进一步把 denoising trajectory 锚定到错误证据。论文证明 discarded token 有信息，但没有彻底解决“informative tentative token”和“misleading tentative token”的区分。

# 15. Limitations

- 最佳 retrieval timestep 是否随问题难度变化仍不清楚。
- tentative token 的可靠性判别仍较粗。
- 对 hallucinated entity 的 emergence / disappearance 模式缺少系统分析。
- 主要验证集中在知识密集、多跳 QA，其他任务是否同样成立需要研究。
- training-free 的优势也意味着没有学习一个最优 retrieval policy。

# 16. 与 Diffusion LLM × Retrieval 的关系

这是 **Retrieval from Diffusion Dynamics** 的标志性论文：不是把普通 RAG 直接搬到 dLLM，而是利用 dLLM 独有的中间并行预测作为检索信号。它证明了 diffusion trajectory 本身就是 retrieval controller 的潜在 state space。

# 17. Idea Mining

| 项目 | 内容 |
|---|---|
| Existing assumption | 低置信、被丢弃 token 没有价值 |
| Observed phenomenon | discarded tentative token 会提前暴露 bridge entity |
| Observable signal | entity emergence、stability、revision、entropy、confidence gap |
| Why it happens | 中间预测已受双向语义约束，但尚未达到 commit 阈值 |
| What remains unexplained | 正确与 hallucinated entity 的动态模式是否不同 |
| Retrieval implication | 可把 trajectory 状态作为 when/what-to-retrieve 的依据 |
| Possible pilot | 逐 timestep 记录实体轨迹并计算 retrieval gain |
| Minimal method if confirmed | 基于 temporal stability 的 retrieval trigger / query filter |

# 18. Pilot Experiment + Motivation Figure

定义某个 timestep 的真实检索增益：

`G_t = Acc(retrieve at t) - Acc(no retrieval)`。

对每个样本记录：entropy、top-2 gap、entity stability、revision count、首次 entity emergence timestep，然后分别预测 `G_t`。

**最值得画的 Figure 1**：横轴为 entity stability 或 revision frequency，纵轴为 retrieval gain；若出现“低稳定/高修订 → 高 retrieval gain”的清晰关系，就得到一个比“entropy 高就检索”更强的 phenomenon。

# 19. 复现价值

- 代码/实现价值：★★★★★
- 训练需求：无
- 2×A100 可行性：★★★★★
- Pilot 成本：低到中
- 作为 baseline：★★★★★
- 作为 idea source：★★★★★

# 20. 读完这篇应该记住什么

1. `low-confidence ≠ no-information`。
2. dLLM tentative token 是一种真正的 lookahead signal。
3. 检索可以发生在最终答案形成之前，而不是只在 question 或已生成 prefix 上进行。
4. 最重要的未解问题是：如何区分真实 bridge entity 与 transient hallucination。
5. 下一步最值得做的不是堆模块，而是先验证 `stability / revision / trajectory uncertainty` 谁真正预测 retrieval gain。
