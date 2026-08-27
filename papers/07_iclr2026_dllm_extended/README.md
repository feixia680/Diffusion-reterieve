# 07 — ICLR 2026 dLLM Extended Reading

本目录补充 **ICLR 2026 正式论文**，目的不是扩大会议覆盖面，而是系统收集能为 `Diffusion LLM × Dynamic Retrieval` 提供 **现象、状态变量、failure mode、控制信号** 的 dLLM 工作。

## A. Revision / Remasking / Flexible Decoding

### Diffusion Language Models are Provably Optimal Parallel Samplers
- PDF: `ICLR2026_Optimal_Parallel_Samplers.pdf`
- 核心现象/理论：允许 **revision / remasking** 的 DLM 在并行采样表达能力和空间复杂度上严格强于禁止修改已生成 token 的版本。
- 对 retrieval 的意义：revision 不只是 decoding trick，而是 dLLM 独有的 temporal signal；可研究 `revision frequency / revision locality → retrieval gain`。

### Beyond Masks: Efficient, Flexible Diffusion Language Models via Deletion-Insertion Processes
- PDF: `ICLR2026_Beyond_Masks_DID.pdf`
- 核心现象：固定 mask canvas 并非唯一 diffusion 生成形式；deletion/insertion 可提供动态长度和自修正。
- Retrieval 迁移：知识注入可能改变所需 reasoning length / evidence slots，可研究 retrieval 是否触发结构扩张或收缩。

### ReFusion: A Diffusion Large Language Model with Parallel Autoregressive Decoding
- PDF: `ICLR2026_ReFusion.pdf`
- 核心现象：纯 token-level 并行容易产生 incoherence；提升到 slot-level 后可兼顾并行与因果局部生成。
- Retrieval 迁移：检索证据可能更适合以 semantic slot / evidence slot 为单位介入，而非全序列统一 guidance。

## B. Convergence / Stability / Adaptive Compute

### SureLock: Stopping Computation for Converged Tokens in Masked Diffusion-LM Decoding
- PDF: `ICLR2026_SureLock.pdf`
- 核心现象：许多 token 在完整 denoising 结束前已经稳定；跨 step posterior stability 可可靠检测这种收敛。
- Retrieval 迁移：`token stability map` 可用于定位尚未收敛、可能缺知识的位置；也可研究 evidence 是否只应作用于 unlocked spans。

### SparseD: Sparse Attention for Diffusion Language Models
- PDF: `ICLR2026_SparseD.pdf`
- 核心现象：DLM attention 具有 **head-specific、跨 denoising step 高稳定性、早期 step 更关键** 的特殊规律。
- Retrieval 迁移：外部 evidence 的有效作用窗口可能强烈依赖 timestep；可测 `retrieval attention mass × timestep × final gain`。

### FlashDLM
- PDF: `ICLR2026_FlashDLM.pdf`
- 核心现象：部分 KV projection 跨 denoising steps 足够稳定，可以复用；同时低 step 推理会引入 token incoherence。
- Retrieval 迁移：retrieved context 加入后哪些 KV / semantic states 被真正改写？可以用 state-change magnitude 判断 evidence 是否有用。

## C. Intermediate-State Causality / Vulnerability

### Toward Safer Diffusion Language Models: Discovery and Mitigation of Priming Vulnerability
- PDF: `ICLR2026_Priming_Vulnerability.pdf`
- 核心现象：**中间 step 出现的 affirmative token 可以持续影响后续 denoising**，即使最终模型本来已安全对齐。
- Retrieval 迁移：这证明 intermediate token 具有因果影响而不仅是“可读 signal”；错误 retrieved entity / fact 是否也会产生类似 priming cascade，值得专门测。

## D. Retrieval Orchestration Control

### DeepRAG: Thinking to Retrieve Step by Step for Large Language Models
- PDF: `ICLR2026_DeepRAG.pdf`
- 虽非 dLLM，但它把 `retrieve vs reason` 建模为逐步决策，是 Dynamic Retrieval 的关键 control baseline。
- dLLM 迁移：可把 state 从普通 reasoning trace 扩展成 `tentative tokens + revision + entropy + stability + timestep`，研究 diffusion-native retrieval policy。

## 建议优先做的现象实验

1. **Revision vs Retrieval Gain**：revision frequency 是否比 instantaneous entropy 更能预测检索收益？
2. **Stability Map vs Knowledge Deficit**：未收敛 token 是否集中对应 entity / relation / number 等知识槽？
3. **Retrieval Timing × Attention Dynamics**：早期 evidence 是否比晚期 evidence 更能改变 trajectory？
4. **Evidence Priming**：错误检索内容若在早期 tentative state 中形成锚点，是否产生持续错误 cascade？
5. **State Change as Utility Signal**：retrieval 前后 hidden/KV/posterior 变化幅度能否预测最终 answer gain？
