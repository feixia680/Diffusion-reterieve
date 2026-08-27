# 摘要（中文翻译）

近期 RAG 能处理复杂 query，但研究往往忽略一个关键挑战：不同 retriever 需要本质不同的 query formulation 才能达到最佳表现。本文首次系统分析 LLM 如何通过 RL 学习适应不同 retriever 的 query formulation。实验发现 RL 确实能让 LLM 针对 retriever 特性调整 query，而且不同 retriever 的最佳 query style 出乎意料地不同，例如 descriptive 与 question-like 风格；因此在一个 retriever 上学到的策略对另一个可能无效。加入 retriever-specific human guidance 和扩大模型规模还能进一步提升表现。为学习多 retrieval-step trajectory，作者提出 branching-based rollout 提升训练稳定性。

# 1. TL;DR

**Query rewriting 是 environment-dependent 的。** 没有一个“ universally better query”。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.2013/
- Code：https://github.com/LCO-Embedding/Envs-aware-Information-Retrieval
- 标签：`Retriever-aware Query Formulation`

# 3. 背景

很多 query rewriting 方法默认改写后可用于任意 retriever。

# 4. Assumption

> 一个更语义完整/详细的 query 对所有 retriever 都更好。

# 5. 核心现象

不同 retriever 最优 query style 显著不同；策略存在跨 retriever transfer failure。

# 6. Why

BM25 偏 lexical overlap，dense 偏 semantic geometry，其他 multi-vector retriever 又有不同 inductive bias。

# 7. Formulation

在 retrieval environment 中通过 RL 学 query policy，使 downstream retrieval reward 最大。

# 8. 方法

retriever-specific RL + human guidance + branching rollout。

# 9. 关键理解

`which retriever` 与 `how to query` 必须联合考虑。

# 10. 流程

state → formulate query for current retriever → retrieve → feedback → next query。

# 11. 实验

多 retriever、多 query styles、多轮 retrieval。

# 12. 主结果

retriever-aware formulation 明显优于 environment-agnostic rewriting。

# 13. Ablation

human guidance、model size、branch rollout、cross-retriever transfer。

# 14. Failure

RL 成本高；retriever/corpus 更新后 policy 可能过时。

# 15. Limitations

没有 dLLM intermediate state，也没有显式考虑 latency/cost。

# 16. dLLM 迁移

SARDI 的 tentative token query 也不应默认适合所有 retriever；不同 state 可能对应不同 query form。

# 17. Idea Mining

测试同一 tentative entity query 在 BM25/dense/ColBERT 上的 optimal formulation 是否随 denoising timestep 改变。

# 18. Pilot

构建 `(state, retriever, query form) → downstream gain` 小矩阵；先观察 interaction，而不是直接训练 RL。

# 19. 复现

★★★★☆；pilot 可低成本完成。

# 20. 记住什么

Retrieval Orchestration 的核心不是只选检索器，还要让 query form 与 environment 对齐。
