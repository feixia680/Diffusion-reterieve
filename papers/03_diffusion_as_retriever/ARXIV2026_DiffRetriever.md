# 摘要（中文翻译）

本文展示如何将扩散语言模型用作高效且有效的检索器。已有 DLM retriever（如 DiffEmbed）沿用 BERT 风格编码，将 query 或 passage 表示成单个 mean-pooled 向量，却忽略了 DLM 的原生训练能力：在双向注意力下对 masked positions 进行预测。DiffRetriever 直接利用这一能力，在 query/passage 后附加一个或多个 `[MASK]` 位置，并把这些位置的输出作为 retrieval representations；所有表示在一次 forward 中获得。单个 mask 时已经优于同 backbone 的 DiffEmbed；多个 mask 时可以自然扩展到类似 ColBERT 的多表示细粒度匹配，而 AR LLM 要获得多个 representation 通常必须顺序解码，延迟更高。DiffRetriever 在匹配比较中取得最强总体效果，并发现训练集上选出的 mask 数能跨数据集迁移，但不同 query 的最优数量存在差异，说明仍有 adaptive allocation 空间。

# 1. TL;DR

dLLM 能在一个 forward 中并行产生 **K 个代表性 retrieval tokens**。最有价值的未解问题不是再固定 K，而是 `query → optimal K`。

# 2. 基本信息

- 题目：DiffRetriever: Parallel Representative Tokens for Retrieval with Diffusion Language Models
- 状态：arXiv 2026
- Code：https://github.com/ielab/diffretriever
- Source：https://arxiv.org/abs/2605.07210

# 3. 背景

单向量 dense retrieval 对复杂/多意图 query 表示不足；multi-vector 方法效果好但编码和索引成本高。

# 4. Existing Assumption

> DLM 做 retrieval 也应该像 BERT 一样 mean pool 成单向量。

# 5. 核心现象

- masked prediction token 本身是强 retrieval representation。
- 多个 mask 可并行产生多表示，几乎不增加 sequential latency。
- 不同 query 的最佳 representation 数量不同。

# 6. Why

不同 masked slots 在双向上下文中可以形成互补语义视角；并行 forward 避免 AR sequential representative generation。

# 7. Problem Formulation

对 query 产生 `K_q` 个向量，对 document 产生 `K_d` 个向量，再做 late interaction；核心预算变量是 K。

# 8. 方法总览

`text + K masks → one DLM forward → K hidden/prediction representations → multi-vector matching`。

# 9. 方法细节

K=1 是 single-representation；K>1 时使用多代表 token 进行细粒度 matching。其优势来自 native masked-position prediction，而不是额外 decoder。

# 10. 流程

离线编码 corpus；在线 query 生成 K 个表示；执行 ANN/late interaction 检索。

# 11. 实验

与 DiffEmbed、PromptReps、RepLLaMA 等 matched baselines 比较 effectiveness 与 encoding latency。

# 12. 主结果

单表示即优于 DiffEmbed；多表示取得更强 aggregate retrieval 效果，且比 AR multi-representation 更高效。

# 13. Ablation

最重要的 ablation 是 K。训练集最优 K 可迁移，但 per-query oracle 更高，直接暴露 adaptive budget gap。

# 14. Failure Case

固定 K 会对简单 query 浪费表示，对复杂 query 又可能不足；multi-vector index 也增加存储/匹配成本。

# 15. Limitations

未真正解决 per-query dynamic K；retrieval representations 如何随 denoising state 变化也未研究。

# 16. 与本方向关系

这是 `Diffusion as Retriever`，可以自然连接 Retrieval Orchestration 的 `how much to retrieve/represent`。

# 17. Idea Mining

Observable：query entropy、semantic multiplicity、entity 数、mask-token diversity。目标：预测 `K*`。

# 18. Pilot

对每个 query 枚举 K=1/2/4/8，得到 oracle K*；再测试 query difficulty、embedding diversity 与 K* 的相关性。

# 19. 复现价值

2×A100：★★★★★；代码开源；pilot 低成本；idea source：★★★★★。

# 20. 记住什么

1. 多个 `[MASK]` 是天然并行 retrieval representatives。
2. 固定 K 不是终点，adaptive K 是最明显 gap。
