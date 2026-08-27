# Diffusion LLM × Dynamic Retrieval (2026)

面向 **Diffusion Language Models (dLLMs) × Retrieval / RAG / Search Agents** 的问题型论文库。

本仓库不按“会议列表”简单堆论文，而按一个更适合找 idea 的原则组织：

> **先观察 dLLM 独有现象 → 找到可测量信号 / failure mode → 再问 retrieval 能否利用或修复它。**

核心研究模板：

```text
Observation / Phenomenon
        ↓
Why does it happen?
        ↓
What signal does diffusion expose that AR LLMs do not?
        ↓
Can retrieval use this signal, or correct this failure?
        ↓
Minimal method + controlled experiment
```

## 目录

```text
papers/
├── 01_core_dynamic_retrieval/
│   ├── ICML2026_SARDI.pdf
│   ├── ARXIV2026_SPREAD.pdf
│   └── ARXIV2026_ARAM.pdf
├── 02_search_agents_tool_use/
│   └── SIGIR2026_DLLM_Searcher.pdf
├── 03_diffusion_as_retriever/
│   ├── ACL2026_Diffusion_Pretrained_Embeddings.pdf
│   ├── ARXIV2026_DiffRetriever.pdf
│   └── ICML2026_R4T_RL_Compiled_Diffusion.pdf
├── 04_dllm_dynamics_for_idea_mining/
│   ├── ICLR2026_Prophet_Early_Answer_Convergence.pdf
│   ├── ICML2026_LookUM.pdf
│   ├── ICML2026_SOAR.pdf
│   ├── ICML2026_DAPD.pdf
│   ├── ICML2026_Residual_Context_Diffusion.pdf
│   ├── ICML2026_CoDiLA.pdf
│   ├── ICML2026_d2_Trajectory_Likelihood.pdf
│   └── ICML2026_PoE_Bridge.pdf
├── 05_high_value_preprints/
│   ├── ARXIV2026_Associative_Memory_DLM.pdf
│   └── ARXIV2026_Introspective_DLM.pdf
├── 06_adjacent_dynamic_retrieval/
│   ├── ACL2026_Adaptive_Retrieval_for_Reasoning.pdf
│   └── ACLFindings2026_QuCo_RAG.pdf
├── 07_iclr2026_dllm_extended/
│   ├── ICLR2026_Optimal_Parallel_Samplers.pdf
│   ├── ICLR2026_FlashDLM.pdf
│   ├── ICLR2026_SureLock.pdf
│   ├── ICLR2026_ReFusion.pdf
│   ├── ICLR2026_SparseD.pdf
│   ├── ICLR2026_Beyond_Masks_DID.pdf
│   ├── ICLR2026_Priming_Vulnerability.pdf
│   └── ICLR2026_DeepRAG.pdf
└── 08_acl2026_dllm_adaptive_retrieval/
    ├── ACL2026_Diffuse_Thinking.pdf
    ├── ACL2026_Denoising_Process_Rewards.pdf
    ├── ACL2026_UNCODE_Decoding_Biases.pdf
    ├── ACL2026_Semantic_Aware_Adaptive_Denoising.pdf
    ├── ACL2026_d_TreeRPO.pdf
    ├── ACL2026_QuDAR.pdf
    ├── ACL2026_SPARKLE.pdf
    ├── ACL2026_R3AG.pdf
    ├── ACL2026_Environment_Aware_IR.pdf
    └── ACL2026_GuarantRAG.pdf
```

PDF 由 `.github/workflows/sync-papers.yml` 从 ICLR Proceedings / ACL Anthology / arXiv 等公开来源自动同步。扩充后目标规模约 **37 篇**，其中新增部分优先使用 ICLR 2026 与 ACL 2026 Main 正式论文。

## 新增两条重点阅读线

### ICLR 2026：研究 dLLM 到底“特殊”在哪里

新增 ICLR 论文重点覆盖：

- **revision / remasking 的表达能力**：`Optimal Parallel Samplers`
- **跨 step 状态稳定与收敛**：`SureLock`
- **attention 跨 denoising step 的规律**：`SparseD`
- **KV/state 可复用性**：`FlashDLM`
- **中间 token 的因果 priming 效应**：`Priming Vulnerability`
- **并行与局部因果结构的折中**：`ReFusion`
- **动态结构/长度生成**：`Beyond Masks / DID`
- **retrieve vs reason 的序贯控制基线**：`DeepRAG`

这组论文主要服务一个问题：

> **哪些 diffusion-specific state 真正能够作为 retrieval trigger / routing signal / stopping signal？**

详见 `papers/07_iclr2026_dllm_extended/README.md`。

### ACL 2026 Main：把现象变成 Retrieval Orchestration

新增 ACL Main 论文分成两组：

**dLLM dynamics**
- Diffuse Thinking
- Denoising Process Rewards
- UNCODE
- Semantic-Aware Adaptive Denoising
- d-TreeRPO

**adaptive retrieval / orchestration**
- QuDAR
- SPARKLE
- R³AG
- Environment-aware Information Retrieval
- GuarantRAG

这组论文对应：

```text
Diffusion state / phenomenon
        ↓
Is retrieval needed?
        ↓
Which retriever?
        ↓
How to formulate query?
        ↓
How much / when / stop?
        ↓
Was evidence actually integrated?
```

详见 `papers/08_acl2026_dllm_adaptive_retrieval/README.md`。

## 2026 年最核心的研究链

### 1. dLLM 的中间状态不是“废计算”

- **SARDI (ICML 2026)**：被丢弃的低置信 tentative tokens 会提前暴露 salient entities，因此可以直接作为 lookahead retrieval signal。
- **Residual Context Diffusion (ICML 2026)**：被 remask / 丢弃的预测仍包含后续 denoising 有用的上下文信息。
- **Priming Vulnerability (ICLR 2026)**：中间 token 甚至可能对后续 trajectory 产生持续因果影响。

这共同说明：**intermediate / discarded states 既有信息，也可能有因果作用。**

### 2. dLLM 在最终解码之前已经暴露“答案状态”

- **Diffusion Language Models Know the Answer Before Decoding (ICLR 2026)**：大量样本在完整 denoising 结束前已经出现 early answer convergence。
- **SureLock (ICLR 2026)**：部分 token posterior 会提前跨 step 稳定。
- **Semantic-Aware Adaptive Denoising (ACL 2026)**：semantic stabilization 可以用于 adaptive computation。

这提示 retrieval 不一定只能在 generation 前触发：可以研究 **knowledge need / evidence sufficiency 是否也会提前收敛**。

### 3. dLLM 的错误高度依赖 trajectory 与 commit order

- **LookUM (ICML 2026)**：greedy confidence-based unmasking 是短视的，错误路径会导致 sequence-level uncertainty 增大。
- **SOAR (ICML 2026)**：低置信时更值得 search，高置信时更适合 aggressive parallel decode。
- **DAPD (ICML 2026)**：是否能同时 unmask 不仅取决于 token confidence，还取决于 token 间 dependency。
- **UNCODE (ACL 2026)**：uncertainty-based decoding 本身会产生系统性 decoding bias。
- **d-TreeRPO / Denoising Process Rewards (ACL 2026)**：trajectory/order 本身需要进入 reward 与 policy evaluation。

这提示动态 retrieval 应从 `instantaneous confidence` 升级为 **trajectory-aware / dependency-aware / calibrated retrieval policy**。

### 4. dLLM 对 retrieval context 的响应有特殊 failure mode

- **SPREAD (2026 preprint)**：RAG 条件下观察到 Response Semantic Drift，答案会在迭代 denoising 中逐步偏离 query 语义。
- **ARAM (2026 preprint)**：retrieved evidence 与 parametric prior 冲突时，固定 retrieval guidance 会伤害生成，需要随 denoising state 自适应调节。
- **GuarantRAG (ACL 2026)**：即使正确 evidence 已经 retrieve，generator 仍可能出现 integration bottleneck。

这条线对应：**Retrieval for Diffusion Dynamics**。

### 5. Retrieval Orchestration 本身正在从单一动作走向联合控制

- **DeepRAG (ICLR 2026)**：逐步决定 retrieve vs reason。
- **R³AG (ACL 2026)**：不同 query 应路由到不同 retriever，且 generation utility 不等于 relevance。
- **QuDAR (ACL 2026)**：retriever type 与 query form 都需要 query-wise adaptive weighting。
- **Environment-aware IR (ACL 2026)**：不同 retriever 偏好的 query formulation 显著不同。
- **SPARKLE (ACL 2026)**：可用独立 proxy controller 控制 retrieval policy。

这为未来的 diffusion-native orchestration 提供直接 baseline：

```text
state_t = tentative tokens + revision + entropy + stability + timestep + reasoning state
        ↓
retrieve / reason / choose retriever / rewrite / top-k / stop
```

### 6. dLLM 的并行 / 任意顺序结构可以改变检索与 Agent 执行方式

- **DLLM-Searcher (SIGIR 2026)**：利用 flexible decoding order，优先生成 tool call，使模型在等待 search response 时继续 reasoning。
- **DiffRetriever (2026 preprint)**：用多个 `[MASK]` 位置一次并行产生多 representative retrieval tokens。
- **R4T (ICML 2026)**：用 diffusion 一次生成 set-valued retrieval outputs，服务 diversity / coverage / complementarity 等集合目标。
- **Diffuse Thinking (ACL 2026)**：dLLM 可高效并行提出 diverse intermediate thoughts。

## 两个研究视角

### Retrieval **from** Diffusion Dynamics

利用 dLLM 独有状态决定：

- when to retrieve
- what to retrieve
- how much to retrieve
- which retriever to use
- whether to retrieve again / stop

可观察信号包括：tentative tokens、entropy、revision、entity stability、mask pattern、attention dependency、trajectory likelihood、posterior stability、semantic convergence、early convergence。

### Retrieval **for** Diffusion Dynamics

用 retrieval 修复：

- semantic drift
- hallucinated / unstable entities
- premature commitment
- retrieval-prior conflict
- noisy evidence sensitivity
- evidence forgetting / dilution during iterative denoising
- integration bottleneck
- early erroneous priming

## 强烈建议的阅读顺序

### 第一阶段：先理解 dLLM 特殊现象
1. `ICLR2026_Prophet_Early_Answer_Convergence.pdf`
2. `ICLR2026_SureLock.pdf`
3. `ACL2026_UNCODE_Decoding_Biases.pdf`
4. `ICML2026_LookUM.pdf`
5. `ICML2026_Residual_Context_Diffusion.pdf`
6. `ICLR2026_Priming_Vulnerability.pdf`

### 第二阶段：看这些 signal 怎么被用于 retrieval
7. `ICML2026_SARDI.pdf`
8. `ARXIV2026_SPREAD.pdf`
9. `ARXIV2026_ARAM.pdf`
10. `SIGIR2026_DLLM_Searcher.pdf`

### 第三阶段：学习 Retrieval Orchestration
11. `ICLR2026_DeepRAG.pdf`
12. `ACL2026_R3AG.pdf`
13. `ACL2026_QuDAR.pdf`
14. `ACL2026_Environment_Aware_IR.pdf`
15. `ACL2026_SPARKLE.pdf`
16. `ACL2026_GuarantRAG.pdf`

核心目的不是复现所有方法，而是建立对 **dLLM denoising trajectory 中什么信号具有预测价值，以及如何把信号转成 retrieval action** 的直觉。

## 状态说明

- `ICLR / ICML / ACL / SIGIR 2026`：已核验正式接收/发表。
- `ACL Findings 2026`：作为相邻动态检索参考。
- `ARXIV2026`：截至 2026-08 尚未把其标成正式顶会论文；保留是因为其现象与本研究线高度相关。

详见 [`papers/README.md`](papers/README.md)、[`papers/07_iclr2026_dllm_extended/README.md`](papers/07_iclr2026_dllm_extended/README.md)、[`papers/08_acl2026_dllm_adaptive_retrieval/README.md`](papers/08_acl2026_dllm_adaptive_retrieval/README.md) 与 [`IDEA_MINING.md`](IDEA_MINING.md)。
