# 摘要（中文翻译）

本文提出 pplx-embed，一组面向 web-scale retrieval 的多语言 embedding 模型，采用扩散预训练语言模型作为 backbone，并进行多阶段对比学习。扩散式预训练带来的双向注意力使模型能够在 passage 内捕获更完整的双向上下文，从而让 mean pooling 更好地保留长文档的全局信息。作者发布 pplx-embed-v1 用于标准检索，以及 pplx-embed-context-v1 用于把全局文档上下文纳入 passage representation 的 contextualized embedding。pplx-embed-v1 在 MTEB Multilingual v2、MTEB Code、BERGEN、ToolRet 等检索基准上取得有竞争力的结果，pplx-embed-context-v1 在 ConTEB 上刷新结果。

# 1. TL;DR

扩散预训练的价值不只在生成：**bidirectional context 本身就是一种强 retrieval representation prior**。

# 2. 基本信息

- 题目：Diffusion-Pretrained Dense and Contextual Embeddings
- 作者：Sedigheh Eslami 等
- 会议：ACL 2026 Industry Track
- Official：https://aclanthology.org/2026.acl-industry.69/
- 标签：`Dense Retrieval`、`Contextual Embedding`

# 3. 背景

很多 decoder-only embedding 需要额外设计 pooling 或特殊 token 才能把单向表示转成检索向量；长文档切块后又容易丢失全局上下文。

# 4. Existing Assumption

> diffusion pretraining 的主要优势是并行生成，与 embedding retrieval 没有直接关系。

# 5. 核心现象

双向 attention 让普通 mean pooling 也能聚合更完整的 passage semantics，并为 contextualized chunk embedding 提供自然基础。

# 6. Why

每个 token representation 同时融合左右文，mean pooling 汇总的是全局条件化 token，而不是偏单向的局部状态。

# 7. Problem Formulation

学习 `f(q)` 与 `f(d)` 的 embedding，使相关 query-document 相似度高；context 版进一步建模 passage 与所属 document 的关系。

# 8. 方法

扩散预训练 backbone + 多阶段 contrastive learning；标准版输出 dense embedding，context 版把全局文档信息注入 passage representation。

# 9. 公式理解

核心仍是 contrastive objective：正样本相似度上升、hard negatives 相似度下降。创新重点不在新 loss，而在 backbone representation 与 context construction。

# 10. 流程

`query/document → diffusion-pretrained encoder → pooling/contextualization → vector → ANN retrieval`。

# 11. 实验

覆盖 MTEB multilingual/code、BERGEN、ToolRet、ConTEB 等，多语言、代码、tool retrieval 和 contextual retrieval 都有验证。

# 12. 主结果

标准模型具有竞争力，contextual 版本在 ConTEB 创下新结果，说明双向预训练对长上下文检索确实有迁移价值。

# 13. Ablation

重点应看 diffusion backbone vs 同规模非 diffusion backbone、mean pooling 与其他 pooling、是否加入 document context。

# 14. Failure Case

它仍是“训练一个 retriever”，无法直接回答 dynamic retrieval 的 when/where 问题。

# 15. Limitations

训练成本高于 training-free 方法；性能提升中有多少来自数据/对比学习、有多少来自 diffusion pretraining，需要严格控制。

# 16. 与本方向关系

它说明 dLLM hidden state 本身可能成为 query/passage representation space，为后续直接从 intermediate denoising state 做 retrieval 提供依据。

# 17. Idea Mining

最值得验证：不同 denoising layer/timestep 的 hidden state 哪个最适合作为 retrieval embedding？是否 query-dependent？

# 18. Pilot

冻结一个 dLLM，抽取不同 layer × timestep 表示，在同一 ANN index 上测 Recall/nDCG，画热图。

# 19. 复现价值

2×A100：★★★☆☆；训练需求：中高；idea source：★★★★☆。

# 20. 记住什么

1. diffusion bidirectionality 可以直接服务 retrieval representation。
2. 但这篇更偏 retriever training，不是 Dynamic Retrieval 本身。
