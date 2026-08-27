# 摘要（中文翻译）

RAG 依赖 retrieval module 为 LLM 提供 grounded evidence。混合 sparse+dense retrieval 虽常优于单检索器，但多数方法使用固定融合权重，忽略 query/corpus 差异；query expansion 也常静态地与原 query 混合，可能引入噪声。QuDAR 从两个视角同时自适应：retriever type（sparse vs dense）和 query format（original vs expanded）。它利用 margin-derived confidence（如 top1-top2 score gap）和盲式 LLM relevance scoring，为每个 query 动态分配权重，在 lexical specificity 与 semantic breadth 之间平衡并抑制扩展噪声。该方法轻量、retriever-agnostic，实验显示相比静态 baseline 持续提升检索质量并使不同 query 上表现更稳定。

# 1. TL;DR

**Which retriever** 和 **How to query** 不应该分开做；它们是同一个 query-dependent orchestration 问题。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.1791/
- 标签：`Hybrid Retrieval`、`Query Expansion`

# 3. 背景

固定 BM25/dense 权重、固定 original/expanded query 权重。

# 4. Assumption

> 同一 fusion strategy 对所有 query 都合适。

# 5. 核心现象

不同 query 对 lexical vs semantic retriever、原始 vs expansion 的偏好明显不同。

# 6. Why

exact entity、长尾专名与语义 paraphrase 的检索需求不同；query expansion 有时提升 recall，有时加入噪声。

# 7. Formulation

为每个 query 学/估计两组权重：retriever perspective + query perspective。

# 8. 方法

margin confidence + blind LLM relevance score → dynamic fusion。

# 9. 关键理解

retrieval orchestration 的 state 不应只有问题文本，还应包含 retriever feedback。

# 10. 流程

original/expanded query × sparse/dense retrieve → confidence/relevance estimate → weighted fusion。

# 11. 实验

多 retrieval benchmark，比较 static hybrid 与 adaptive fusion。

# 12. 主结果

一致提升 overall retrieval quality，query-level variance 更稳定。

# 13. Ablation

只 adaptive retriever、只 adaptive query、双视角联合。

# 14. Failure

margin confidence 也可能 calibration 不良；LLM relevance judge 有额外成本。

# 15. Limitations

主要是单轮 retrieval，没有 reasoning trajectory state。

# 16. dLLM 迁移

把 tentative entities/revision 加入 state，可做 `denoising state → retriever + query form` 联合控制。

# 17. Idea Mining

同一 tentative entity 在 BM25 与 dense 上可能表现不同；entity stability 是否能预测 retriever preference？

# 18. Pilot

逐 query 计算 BM25/dense 的真实 downstream gain，测试 dLLM state features 能否预测最佳 retriever。

# 19. 复现

★★★★★；非常适合 2×A100。

# 20. 记住什么

Routing 与 rewriting 应联合研究，而不是两个独立模块。
