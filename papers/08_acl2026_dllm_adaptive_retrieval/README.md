# 08 — ACL 2026 dLLM + Adaptive Retrieval / Orchestration

本目录只补充 **ACL 2026 Main Long Papers**，分成两组：

- dLLM 独特 dynamics：用于寻找 diffusion-specific observation；
- adaptive retrieval / orchestration：用于学习如何把 observation 变成 retrieval action。

## A. dLLM Dynamics / Reasoning

### Diffuse Thinking: Exploring Diffusion Language Models as Efficient Thought Proposers for Reasoning
- PDF: `ACL2026_Diffuse_Thinking.pdf`
- Observation：dLLM 的并行非自回归生成适合一次提出多个 diverse intermediate thoughts，而 AR 模型更适合评估。
- Retrieval 价值：tentative thought diversity 是否能够产生 **multi-query / multi-evidence exploration signal**？

### Advancing Reasoning in Diffusion Language Models with Denoising Process Rewards
- PDF: `ACL2026_Denoising_Process_Rewards.pdf`
- Observation：只看 final outcome 会丢失 denoising trajectory 中不同区间对结果的贡献；可为中间区间定义 process-level contribution。
- Retrieval 价值：可以定义 `retrieval at step t` 对最终结果的 marginal contribution，寻找真正有效的 retrieval window。

### Empirical Analysis of Decoding Biases in Masked Diffusion Models (UNCODE)
- PDF: `ACL2026_UNCODE_Decoding_Biases.pdf`
- Observation：常见 uncertainty-based decoding 会产生 **rigid boundary bias** 与 **trivial token bias**。
- Retrieval 价值：如果 entropy/confidence 本身带 decoding bias，就不能直接把它当 retrieval trigger；需要比较校准后信号与真实 retrieval gain。

### Towards Efficient and Effective Diffusion Language Model Inference via Semantic-Aware Adaptive Denoising
- PDF: `ACL2026_Semantic_Aware_Adaptive_Denoising.pdf`
- Observation：token 的 semantic stabilization 可以早于完整 denoising 结束，单纯固定步数造成浪费。
- Retrieval 价值：semantic convergence / non-convergence 可以作为 evidence sufficiency、retrieve-again 或 stop 的候选信号。

### d-TreeRPO: Towards More Reliable Policy Optimization for Diffusion Language Models
- PDF: `ACL2026_d_TreeRPO.pdf`
- Observation：dLLM 的不同 decoding orders / trajectories 会造成 reward estimation 与 policy optimization 偏差。
- Retrieval 价值：如果 retrieval action 嵌入 denoising trajectory，评价 policy 时也需要显式考虑 trajectory / order，而不能只看 final answer。

## B. Adaptive Retrieval / Orchestration

### QuDAR: Query-Wise Dual-Perspective Adaptive Retrieval
- PDF: `ACL2026_QuDAR.pdf`
- Observation：固定 sparse/dense fusion 权重和固定 original/expanded query 权重都不可靠，最优组合随 query 改变。
- Orchestration 价值：同时控制 **which retriever + which query form**；非常适合作为 diffusion-state-conditioned routing baseline。

### SPARKLE: A Structured and Plug-and-play Agentic Retrieval Policy for Adaptive RAG Models
- PDF: `ACL2026_SPARKLE.pdf`
- Observation：retrieval decision 可以交给独立 proxy controller，而无需修改主 LLM/retriever。
- Orchestration 价值：适合低算力实验；可把 proxy 输入换成 dLLM trajectory features，验证 diffusion-specific state 是否带来增益。

### R³AG: Retriever Routing for Retrieval-Augmented Generation
- PDF: `ACL2026_R3AG.pdf`
- Observation：retrieval relevance 与 generator utility 不等价，不同 query 对 retriever 的 preference 会发生变化。
- Orchestration 价值：未来 dLLM routing 不应只优化 Recall，而应测 **evidence 对后续 denoising/final answer 的真实 utility**。

### Understanding the Behaviors of Environment-aware Information Retrieval
- PDF: `ACL2026_Environment_Aware_IR.pdf`
- Observation：不同 retriever 的最佳 query formulation style 显著不同，在 retriever A 学到的 rewriting strategy 未必迁移到 B。
- Orchestration 价值：`retriever selection` 与 `query formulation` 应联合决策；dLLM tentative states 可能提供更早的 query intent / entity signal。

### GuarantRAG: Guaranteeing Knowledge Integration with Joint Decoding for RAG
- PDF: `ACL2026_GuarantRAG.pdf`
- Observation：即使 retrieve 到正确证据，generator 仍可能因 parametric knowledge conflict 而不使用证据，即 **integration bottleneck**。
- Diffusion 价值：可研究 external evidence 与 dLLM parametric prior 的冲突如何沿 denoising trajectory 演化，以及何时介入最有效。

## 这组 ACL 论文如何和 dLLM 主线拼起来

最值得对照的关系是：

```text
UNCODE / Semantic-Aware Denoising
        ↓
哪些 diffusion signal 可信？
        ↓
R³AG / QuDAR / SPARKLE / Environment-aware IR
        ↓
如何把信号转成 retrieval action？
        ↓
GuarantRAG
        ↓
retrieved evidence 是否真的被模型使用？
```

因此后续 pilot 不应只测 “加 retrieval 后准确率是否提高”，而应拆成：

1. state signal 能否预测 retrieval need；
2. state signal 能否预测 retriever/query preference；
3. retrieved evidence 是否改变 denoising trajectory；
4. trajectory change 是否最终带来 answer gain；
5. 如果没 gain，是 retrieval failure 还是 integration failure。
