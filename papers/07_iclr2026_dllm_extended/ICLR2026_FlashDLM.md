# 摘要（中文翻译）

DLM 具有并行生成和双向建模优势，但 Dream-7B、LLaDA-8B 等 SOTA 模型推理仍很慢，因为迭代 denoising 要多次执行 full-sequence forward，长 prompt/context 下代价尤其高；同时并行生成会产生 token incoherence，减少 denoising steps 又会明显掉质量。FlashDLM 提出两个 training-free 技术：FreeCache 复用跨 denoising steps 已稳定的 KV projection，减少重复计算；Guided Diffusion 使用轻量预训练 AR 模型监督 token unmasking，从而减少总 denoising iterations。多个 reasoning benchmark 上两者结合平均得到约 12.14× 端到端加速，准确率几乎不损失，使 DLM 延迟首次达到甚至超过常用 AR 模型。

# 1. TL;DR

**跨 step 有大量稳定 state 可缓存；不稳定 token 才值得继续算。** 这个观察可直接迁移到“稳定知识无需检索”。

# 2. 基本信息

- ICLR 2026
- Code：https://github.com/ZhanqiuHu/flash-dlm-experimental
- 标签：`KV stability`、`Guided decoding`

# 3. 背景

dLLM 每步重算全序列，KV cache 复用困难。

# 4. Assumption

> 双向 denoising 中所有 KV 都必须每步重新计算。

# 5. 现象

大量 token/KV projection 跨 steps 已稳定，可安全复用；early aggressive skipping 会伤害质量。

# 6. Why

随着 trajectory 收敛，许多 token contextual representation 变化很小。

# 7. Formulation

检测 stable states → cache；用 AR guidance 选择更可靠 unmask。

# 8. 方法

FreeCache + Guided Diffusion。

# 9. 关键理解

这是 compute orchestration：把计算资源集中在真正还在变化的位置。

# 10. 流程

monitor stability → reuse KV → AR-guided commit → fewer steps。

# 11. 实验

Dream-7B、LLaDA-8B，长上下文与 reasoning。

# 12. 结果

平均 12.14× 加速，质量基本保持。

# 13. Ablation

FreeCache 与 Guided Diffusion 单独/组合；cache threshold；AR guide 大小。

# 14. Failure

缓存“表面稳定但语义应被后续 evidence 改写”的 state 可能阻碍纠错。

# 15. Limitations

依赖辅助 AR；关注 latency 而非 knowledge grounding。

# 16. Retrieval 关系

可研究 `state stability → no retrieve`，把计算缓存思想变成 knowledge-action caching。

# 17. Idea Mining

稳定 token 是否在 retrieval 后几乎不变？不稳定 token 是否集中承载 external knowledge need？

# 18. Pilot

统计 token hidden/KV stability 与 retrieval-induced revision 的关系。

# 19. 复现

★★★★☆；系统工程略高。

# 20. 记住什么

跨 step stability 是 dLLM 很强的 observable signal。
