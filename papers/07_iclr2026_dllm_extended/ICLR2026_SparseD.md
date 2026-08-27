# 摘要（中文翻译）

DLM 的推理延迟很大一部分来自 attention 对 context length 的二次复杂度。AR 中已有固定稀疏 attention pattern，但 DLM 表现出不同稀疏行为：不同 head 的 attention pattern 不同；同一 head 的 pattern 在不同 denoising steps 之间高度相似；早期 denoising steps 对生成尤其关键。因此直接套用 AR sparse attention 不合适。SparseD 基于这些观察，只需一次预计算 head-specific sparse pattern，并在后续所有 steps 复用；同时早期使用 full attention，后期再切到 sparse attention。64k context、1024 denoising steps 下，相比 FlashAttention 可获得最高约 1.50× 无损加速。

# 1. TL;DR

**attention structure 跨 denoising step 高度稳定，但早期步骤更敏感。**

# 2. 基本信息

- ICLR 2026
- Code：https://github.com/INV-WZQ/SparseD
- 标签：`Attention dynamics`、`Long context`

# 3. 背景

AR sparse attention 通常假设 pattern 稳定且跨 head 可统一。

# 4. Assumption

> AR 的 sparse pattern 可直接迁移到 DLM。

# 5. 核心现象

1. head-specific；2. cross-step stable；3. early steps critical。

# 6. Why

不同 head 承担不同依赖，而 denoising 后期主要在已有全局结构上 refinement。

# 7. Formulation

早期 full attention，后期重用一次得到的 head-specific sparse mask。

# 8. 方法

SparseD。

# 9. 关键理解

trajectory 的不同阶段具有不同 information sensitivity。

# 10. 流程

early full → estimate patterns → later sparse reuse。

# 11. 实验

长上下文，64k，1024 steps。

# 12. 结果

最高约 1.50× speedup，lossless quality。

# 13. Ablation

switch timestep、sparsity、head-specific vs shared pattern。

# 14. Failure

过早 sparse 会明显破坏 generation。

# 15. Limitations

主要是系统优化，不直接研究 semantic knowledge。

# 16. Retrieval 关系

强提示 **retrieval timing 也可能 early-sensitive**：早期 evidence 改变全局结构，晚期 evidence 可能只能局部修补。

# 17. Idea Mining

测同一 evidence 在 early/mid/late 注入时 attention redistribution 与最终 gain。

# 18. Pilot

画 `retrieval timestep → attention shift → answer gain` 三者关系。

# 19. 复现

★★★★☆；长上下文显存需求较高。

# 20. 记住什么

DLM timestep 不是同质的；任何 retrieval policy 都应显式考虑阶段差异。
