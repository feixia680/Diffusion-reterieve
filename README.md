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
└── 06_adjacent_dynamic_retrieval/
    ├── ACL2026_Adaptive_Retrieval_for_Reasoning.pdf
    └── ACLFindings2026_QuCo_RAG.pdf
```

PDF 由 `.github/workflows/sync-papers.yml` 从 arXiv / ACL Anthology 等公开来源自动同步。

## 2026 年最核心的研究链

### 1. dLLM 的中间状态不是“废计算”

- **SARDI (ICML 2026)**：被丢弃的低置信 tentative tokens 会提前暴露 salient entities，因此可以直接作为 lookahead retrieval signal。
- **Residual Context Diffusion (ICML 2026)**：被 remask / 丢弃的预测仍包含后续 denoising 有用的上下文信息。

这两篇共同说明：**discarded / tentative states 本身含有可利用信息。**

### 2. dLLM 在最终解码之前已经暴露“答案状态”

- **Diffusion Language Models Know the Answer Before Decoding (ICLR 2026)**：大量样本在完整 denoising 结束前已经出现 early answer convergence。

这提示 retrieval 不一定只能在 generation 前触发：可以研究 **knowledge need / answer state 是否也会提前显现**。

### 3. dLLM 的错误高度依赖 trajectory 与 commit order

- **LookUM (ICML 2026)**：greedy confidence-based unmasking 是短视的，错误路径会导致 sequence-level uncertainty 增大。
- **SOAR (ICML 2026)**：低置信时更值得 search，高置信时更适合 aggressive parallel decode。
- **DAPD (ICML 2026)**：是否能同时 unmask 不仅取决于 token confidence，还取决于 token 间 dependency。

这提示动态 retrieval 可以从 `instantaneous confidence` 升级为 **trajectory-aware / dependency-aware retrieval policy**。

### 4. dLLM 对 retrieval context 的响应有特殊 failure mode

- **SPREAD (2026 preprint)**：RAG 条件下观察到 Response Semantic Drift，答案会在迭代 denoising 中逐步偏离 query 语义。
- **ARAM (2026 preprint)**：retrieved evidence 与 parametric prior 冲突时，固定 retrieval guidance 会伤害生成，需要随 denoising state 自适应调节。

这条线对应：**Retrieval for Diffusion Dynamics**。

### 5. dLLM 的并行 / 任意顺序结构可以改变检索与 Agent 执行方式

- **DLLM-Searcher (SIGIR 2026)**：利用 flexible decoding order，优先生成 tool call，使模型在等待 search response 时继续 reasoning。
- **DiffRetriever (2026 preprint)**：用多个 `[MASK]` 位置一次并行产生多 representative retrieval tokens。
- **R4T (ICML 2026)**：用 diffusion 一次生成 set-valued retrieval outputs，服务 diversity / coverage / complementarity 等集合目标。

## 两个研究视角

### Retrieval **from** Diffusion Dynamics

利用 dLLM 独有状态决定：

- when to retrieve
- what to retrieve
- how much to retrieve
- which retriever to use
- whether to retrieve again / stop

可观察信号包括：tentative tokens、entropy、revision、entity stability、mask pattern、attention dependency、trajectory likelihood、early convergence。

### Retrieval **for** Diffusion Dynamics

用 retrieval 修复：

- semantic drift
- hallucinated / unstable entities
- premature commitment
- retrieval-prior conflict
- noisy evidence sensitivity
- evidence forgetting / dilution during iterative denoising

## 强烈建议的阅读顺序

1. `ICLR2026_Prophet_Early_Answer_Convergence.pdf`
2. `ICML2026_Residual_Context_Diffusion.pdf`
3. `ICML2026_LookUM.pdf`
4. `ICML2026_DAPD.pdf`
5. `ICML2026_SARDI.pdf`
6. `ARXIV2026_SPREAD.pdf`
7. `ARXIV2026_ARAM.pdf`
8. `SIGIR2026_DLLM_Searcher.pdf`
9. `ARXIV2026_DiffRetriever.pdf`
10. `ICML2026_R4T_RL_Compiled_Diffusion.pdf`

核心目的不是复现十个方法，而是建立对 **dLLM denoising trajectory 中什么信号具有预测价值** 的直觉。

## 状态说明

- `ICLR / ICML / ACL / SIGIR 2026`：已核验正式接收/发表。
- `ACL Findings 2026`：顶会 Findings，用作相邻动态检索参考。
- `ARXIV2026`：截至 2026-08 尚未把其标成正式顶会论文；保留是因为其现象与本研究线高度相关。

详见 [`papers/README.md`](papers/README.md) 与 [`IDEA_MINING.md`](IDEA_MINING.md)。
