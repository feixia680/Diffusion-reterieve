# 摘要（中文翻译）

dLLM 的并行解码很困难，因为每个 denoising step 只给出 token-wise marginal distribution，而同时 unmask 多个 token 必须考虑 token 之间的依赖。DAPD 是简单、training-free 的依赖感知并行解码方法，利用 self-attention 在 masked token 间构造 conditional dependency graph。图中边表示强 token interaction，无边表示弱依赖；并行解码被转化为在图上选择 independent set，并同时 unmask 所选 token，从而避免同时更新强耦合位置。LLaDA 与 Dream 实验表明 DAPD 改善 accuracy-steps tradeoff，并产生更全局分布的并行更新。

# 1. TL;DR

**token uncertainty 不是独立的。** 真正决定哪些位置能一起更新的是 dependency structure。

# 2. 基本信息

- ICML 2026
- Source：https://arxiv.org/abs/2603.12996
- 标签：`Dependency graph`、`Attention`

# 3. 背景

许多 parallel decoding 方法只按每个 token 的 confidence 独立排序。

# 4. Assumption

> 各 masked positions 可用独立 marginal confidence 判断。

# 5. 现象

强依赖 token 同时更新会互相破坏，单 token marginal 无法暴露这一风险。

# 6. Why

语言结构是关系性的：entity-relation、代码语法、数字表达都存在 joint constraints。

# 7. Formulation

attention → dependency graph `G_t`；选择 independent set 作为可并行更新 positions。

# 8. 方法

从 self-attention 直接推 dependency，无需额外训练或 verifier。

# 9. 关键公式理解

边权可视为位置 i 对 j 的条件影响强度；目标是在保持弱依赖的前提下最大化一次更新数量。

# 10. 流程

forward → build graph → independent set → parallel unmask → next step。

# 11. 实验

LLaDA、Dream；评价 accuracy vs denoising steps。

# 12. 结果

比仅 confidence 的并行策略具有更优 trade-off。

# 13. Ablation

attention layer/head、dependency threshold、independent-set policy。

# 14. Failure

attention 不等价于 causal dependency；某些隐含语义关系未必在单层 attention 中稳定反映。

# 15. Limitations

主要优化 decoding，并未验证 dependency 是否能表示 knowledge deficit。

# 16. Retrieval 关系

启发 **localized retrieval**：模型可能不是“整个问题不知道”，而是某个强依赖 semantic subgraph 缺知识。

# 17. Idea Mining

把 unstable token dependency graph 与 retrieved evidence entity graph 对齐，观察 retrieval 后哪部分依赖结构稳定下来。

# 18. Pilot

标注 entity/relation/date slots，计算每类 slot 的 dependency + uncertainty 与 retrieval gain。

# 19. 复现

★★★★★；训练免；idea source ★★★★★。

# 20. 记住什么

1. uncertainty 具有结构。
2. retrieval need 也可能是结构化而非全局标量。
