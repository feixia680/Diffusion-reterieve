# 摘要（中文翻译）

扩散语言模型（DLM）近年来在自然语言处理任务中展现出很强的能力，但由于 DLM 与传统自回归 LLM 在解码机制上存在根本差异，检索增强生成（RAG）在 DLM 上的潜力仍缺乏系统研究。本文系统测试 DLM 在 RAG 框架中的表现，发现 DLM+RAG 对上下文信息具有更强依赖，但生成精度受到限制。作者识别出关键问题 **Response Semantic Drift（RSD）**：随着去噪迭代进行，生成内容会逐步偏离原始 query 的语义，从而产生低精度回答。作者进一步将这一问题追溯到 DLM 的去噪策略——迭代过程不能持续维持 query 与 response 的语义对齐。为此提出 SPREAD（Semantic-Preserving Retrieval-Augmented Diffusion），通过 query-relevance-guided denoising 主动约束去噪轨迹，使生成始终锚定于 query 语义。实验表明，SPREAD 能显著提高 RAG 场景下的生成精度并缓解语义漂移。

# 1. TL;DR

论文先发现 **RAG 并不会天然解决 dLLM 的事实问题，反而会暴露 Response Semantic Drift**；因此方法不是继续“加更多文档”，而是在 denoising 中持续维持 query relevance。

# 2. 论文基本信息

- 题目：Unlocking the Potentials of Retrieval-Augmented Generation for Diffusion Language Models
- 状态：arXiv 2026
- Source：https://arxiv.org/abs/2601.11342
- 标签：`Retrieval for Diffusion Dynamics`、`Semantic Drift`、`RAG`
- 训练需求：以 inference/denoising 机制改造为核心

# 3. 研究背景

普通 RAG 默认一个重要前提：只要把相关 evidence 放进 context，生成器就能持续围绕 query 使用它。但 dLLM 不是从左到右一次性生成，而是反复修改整个序列；因此“每一步都还能看到问题”并不等于“trajectory 一直和问题语义对齐”。

# 4. Existing Assumption

> 外部 evidence 加入后，迭代去噪会自然利用该 evidence，并逐步逼近正确答案。

SPREAD 发现实际并非如此。

# 5. 核心现象 / Pilot Observation

- DLM+RAG 对 context 的依赖强，但回答 precision 仍可能较低。
- 生成序列在 denoising 过程中会逐步偏离 query 语义，即 RSD。
- 问题不是单纯检索 recall 不够，而是 **evidence 已经存在，trajectory 仍会漂移**。

# 6. 为什么会出现？

去噪目标主要要求恢复合理 token，并没有显式保证“每个 timestep 的全局语义都持续和 query 对齐”。局部预测可能都合理，但经过多步 revision 后，全局语义方向逐渐偏移。

# 7. Problem Formulation

可将问题写成：给定 query `q`、retrieved context `D` 与中间状态 `z_t`，希望后续状态不仅具有高模型概率，还保持高 `Rel(z_t,q)`。核心变成 trajectory-level semantic alignment，而不只是终点答案概率。

# 8. 方法总览

SPREAD 在每个或关键 denoising 阶段评估 query relevance，并用相关性信号引导 token 更新，使 trajectory 在迭代中持续朝 query semantics 收敛。

# 9. 方法详细解析

关键思想是把原本只受模型分布控制的 denoising，增加一项 query-relevance guidance。其作用类似“语义锚”：如果候选更新使回答与 query 更偏离，则降低该更新的优先级；如果更贴近 query，则增强。

# 10. 推理流程

`query → retrieve evidence → initialize masked answer → iterative denoising → relevance-aware guidance → final answer`。

# 11. 实验设置

论文围绕知识密集型 RAG 任务比较普通 DLM+RAG 与加入 semantic-preserving denoising 的方案，重点关注 precision 与 RSD，而非只比较 Recall@k。

# 12. 主实验结果

SPREAD 显著提高回答 precision，并系统缓解 RSD，说明 DLM-RAG 的关键瓶颈之一发生在 **evidence integration during trajectory**，而不是只发生在 retrieval 阶段。

# 13. Ablation / Controlled Experiments

最值得复现：不同 denoising timestep 的 query-response similarity 曲线；有无 guidance 的 drift 曲线；clean / noisy evidence 下 drift 是否变化。

# 14. Failure Cases

若 retrieved evidence 本身相关但错误，强行维持 evidence/query alignment 可能把 trajectory 更稳定地锚定到错误事实。因此“防 drift”与“识别错误 evidence”应联合考虑。

# 15. Limitations

- RSD 的因果机制仍可进一步细分到 entity / relation / numeric span。
- 未充分回答 drift 从何时开始不可逆。
- query relevance 高不等于 factual correctness 高。

# 16. 与 Diffusion LLM × Retrieval 的关系

这是典型 **Retrieval for Diffusion Dynamics**：retrieval 不只是供知识，还可以成为 trajectory anchor。

# 17. Idea Mining

| 项目 | 内容 |
|---|---|
| Assumption | 有 evidence 就会被稳定利用 |
| Phenomenon | response semantics 随 denoising 漂移 |
| Signal | query-state similarity、entity drift、revision |
| Unexplained | 哪类 span 最先漂移、何时不可逆 |
| Retrieval implication | retrieval 可以用于纠偏，而非只用于补知识 |
| Pilot | 逐 step 测 query/evidence/answer 三方相似度 |

# 18. Pilot + Motivation Figure

画 `timestep → semantic alignment` 曲线，并按最终正确/错误样本分组。如果错误样本在某个阶段出现明显拐点，可进一步研究“在拐点前 retrieval 是否更有效”。

# 19. 复现价值

- 2×A100：★★★★★
- Pilot 成本：低
- idea source：★★★★★
- baseline 价值：★★★★☆

# 20. 记住什么

1. RAG evidence 存在不代表 dLLM trajectory 会正确使用它。
2. dLLM 有独特的 Response Semantic Drift。
3. 最值得研究的是 drift 的出现时刻、位置结构与可逆性。
