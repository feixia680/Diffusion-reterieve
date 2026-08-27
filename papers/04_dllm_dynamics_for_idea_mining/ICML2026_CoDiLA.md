# 摘要（中文翻译）

DLM 的并行能力依赖同时预测多个 token，但标准 DLM 从条件 marginal 独立采样，无法建模并发 token 的 joint dependency，容易导致语法不一致和多 token 结构破裂。CoDiLA（Coherent Diffusion with Local Autoregression）通过把并行采样与局部依赖建模结合来解决该问题：不强迫 DLM 负责细粒度 syntax，而是让一个很小的辅助 AR 模型在 diffusion latent 上执行局部顺序解码，从而在 block 之间保持 DLM 的双向与并行能力，同时在 block 内确保 sequential validity。仅使用约 0.6B 的辅助 AR 模型就能显著消除 coherence artifacts，并在代码生成上形成新的准确率-速度 Pareto frontier。

# 1. TL;DR

**并行 marginal 不等于 joint coherence。** 有些局部结构必须联合处理。

# 2. 基本信息

- ICML 2026
- Source：https://arxiv.org/abs/2603.20216
- 标签：`Local dependency`、`Hybrid AR-Diffusion`

# 3. 背景

并行越激进，代码/语法等强耦合结构越容易断裂。

# 4. Assumption

> 每个位置单独预测正确即可得到联合正确序列。

# 5. 现象

同时生成的 token 会出现局部 coherence artifact。

# 6. Why

marginal distributions 缺失联合约束。

# 7. Formulation

把全局问题分为 inter-block parallel generation 与 intra-block sequential consistency。

# 8. 方法

DLM 负责全局 block，tiny AR 负责局部 block 内结构。

# 9. 方法理解

这是“把不同依赖尺度交给不同机制”，而不是强行用一种 decoder 解决所有依赖。

# 10. 流程

DLM latent proposal → local AR infill → next blocks。

# 11. 实验

主要代码生成，关注 syntax/coherence 与 speed。

# 12. 结果

0.6B 辅助模型已能显著修复并行 coherence。

# 13. Ablation

block size、auxiliary model size、local AR 范围。

# 14. Failure

过多依赖 AR 会失去 diffusion 并行优势。

# 15. Limitations

对知识检索的联系是间接的。

# 16. Retrieval 关系

启发 evidence 也存在 joint structure：多个 retrieved facts 不能独立塞入，需要 relation-aware integration。

# 17. Idea Mining

研究 retrieval 后哪些 token cluster 必须联合修正；可以结合 DAPD dependency graph。

# 18. Pilot

比较单 fact retrieval 与成组 relation-consistent evidence 对强依赖 span 的 revision。

# 19. 复现

★★★☆☆；idea source ★★★☆☆。

# 20. 记住什么

局部依赖是 dLLM parallelism 的关键瓶颈，也可能成为 localized evidence integration 的切入口。
