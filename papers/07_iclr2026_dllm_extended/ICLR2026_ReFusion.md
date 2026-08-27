# 摘要（中文翻译）

AR 模型受顺序推理速度限制，而 MDM 虽能并行，却有两个关键问题：双向 attention 阻碍 KV cache，且学习不可处理的 token combination space 容易产生 incoherent generation。ReFusion 把 sequence reorganization 引入 causal attention，将 parallel decoding 从 token-level 提升到更高的 slot-level：slot 之间做 diffusion-based selection，slot 内做 autoregressive infilling，并在每次迭代后把新生成 slot 重排到剩余 mask 前。这样既能完整复用 KV cache，又把学习复杂度从 token combination space 降到更可控的 slot permutation space。七个 benchmark 上，ReFusion 相比 prior MDM 平均性能提升约 34%、速度提升超过 18×，同时接近强 AR 性能并保持平均约 2.33× 加速。

# 1. TL;DR

不是所有 parallelism 都应在 token 粒度；**slot-level structure** 可以同时保留全局并行和局部因果一致性。

# 2. 基本信息

- ICLR 2026
- 标签：`Slot decoding`、`Hybrid causal/diffusion`

# 3. 背景

纯 token-wise diffusion 组合空间巨大，容易 coherence failure。

# 4. Assumption

> 最细粒度 token 就是最佳并行单位。

# 5. 现象

过细并行导致组合学习困难；更高层 slot 结构更稳定。

# 6. Why

自然语言/推理本身具有 phrase/entity/step 层级。

# 7. Formulation

inter-slot diffusion selection + intra-slot AR infill。

# 8. 方法

sequence reorganization 让已生成 slots 成为 causal prefix，从而完整 KV cache。

# 9. 关键理解

把 dependency 尺度显式分层。

# 10. 流程

select slot positions → AR infill slot → reorder → next diffusion iteration。

# 11. 实验

七个 benchmark，quality + latency。

# 12. 结果

对 prior MDM 34% performance gain、>18× speedup；对强 AR 仍约 2.33× 快。

# 13. Ablation

slot size、reordering、AR infill、cache reuse。

# 14. Failure

slot boundary 若与真实语义单位错位，会影响质量。

# 15. Limitations

结构变更较大，不适合直接做 training-free RAG baseline。

# 16. Retrieval 关系

检索 query 也许应该基于 semantic slot，而非散乱 tentative token；尤其 entity-relation slot。

# 17. Idea Mining

从 dLLM trajectory 自动发现 stable semantic slots，再按 slot 触发 localized retrieval。

# 18. Pilot

比较 token-level tentative query 与 entity/phrase slot-level query 的 retrieval precision。

# 19. 复现

★★☆☆☆；方法复杂；idea source ★★★★☆。

# 20. 记住什么

结构化的中间单位可能比单 token 更适合作为 retrieval state。
