# 2026 Paper Catalog — Diffusion LLM × Dynamic Retrieval

> 更新时间：2026-08-27。只把已经核验的正式 venue 写成 `ICLR/ICML/ACL/SIGIR 2026`；尚未核验正式 venue 的工作统一标成 `arXiv 2026`。

## A. Core dLLM × Dynamic Retrieval

| Paper | Venue | 观察到的现象 | Retrieval 价值 | 推荐度 |
|---|---|---|---|---:|
| **Self-Augmenting Retrieval for Diffusion Language Models (SARDI)** | **ICML 2026** | denoising 中最终被丢弃的低置信 tentative tokens 仍会很早暴露关键实体 | 把 intermediate predictions 直接转成 lookahead retrieval query；training-free / retriever-agnostic | ★★★★★ |
| **Unlocking the Potentials of Retrieval-Augmented Generation for Diffusion Language Models (SPREAD)** | arXiv 2026 | DLM + RAG 对 context 依赖更强，但生成会出现 **Response Semantic Drift** | retrieval 不只是“给知识”，还可以充当 denoising trajectory 的 semantic anchor | ★★★★★ |
| **Adaptive Guidance for Retrieval-Augmented Masked Diffusion Models (ARAM)** | arXiv 2026 | noisy / conflicting context 会造成 retrieval-prior conflict；固定 guidance 不可靠 | 根据 denoising state 与 retrieval-induced distribution shift 动态调节 evidence influence | ★★★★★ |

### SARDI

- PDF: `01_core_dynamic_retrieval/ICML2026_SARDI.pdf`
- Source: https://arxiv.org/abs/2606.06474
- **Phenomenon first**: low-confidence token ≠ no-information token。
- 关键 observation：早期 tentative tokens 即使最终被 remask，也经常比 question-only query 更早暴露 multi-hop bridge entity。
- 方法只是 observation 的自然结果：将这些 lookahead tokens 组成动态 retrieval query，再把证据反馈给后续 denoising。
- **后续现象问题**：
  - hallucinated entity 与真实 bridge entity 的 emergence / stability pattern 是否不同？
  - `entity stability`、`revision frequency`、`entropy` 哪个更能预测 retrieval gain？
  - earliest useful retrieval timestep 是否随问题难度系统变化？

### SPREAD

- PDF: `01_core_dynamic_retrieval/ARXIV2026_SPREAD.pdf`
- Source: https://arxiv.org/abs/2601.11342
- **Phenomenon first**: DLM 在 RAG 条件下会出现 Response Semantic Drift，随着 denoising 迭代逐步偏离 query 原始语义。
- 解释：迭代 denoising 并不会自动保证 query–response semantic alignment 持续增强。
- 方法：query-relevance-guided denoising，持续把 trajectory 拉回 query semantics。
- **后续现象问题**：
  - drift 从哪个 timestep 开始变得不可逆？
  - retrieval evidence 能否降低 drift，还是 noisy evidence 反而放大 drift？
  - semantic drift 是否集中发生在 entity / relation / numeric span？

### ARAM

- PDF: `01_core_dynamic_retrieval/ARXIV2026_ARAM.pdf`
- Source: https://arxiv.org/abs/2603.17677
- **Phenomenon first**: retrieved context 并非越强越好，冲突/噪声 context 在不同 denoising state 的伤害程度不同。
- 方法：根据 retrieval-induced distribution shift 的 SNR 动态调整 guidance scale。
- **后续现象问题**：
  - context reliability 与 optimal guidance timestep 的关系？
  - conflict 在早期 denoising 和晚期 denoising 哪个更危险？
  - model parametric prior 与 external evidence 的冲突是否可以从 revision trajectory 中提前识别？

---

## B. dLLM Search Agents / Tool Use

| Paper | Venue | 观察到的现象 | Retrieval / Agent 价值 | 推荐度 |
|---|---|---|---|---:|
| **DLLM-Searcher: Adapting Diffusion Language Model for Efficient Search Agents** | **SIGIR 2026 Full Paper** | ReAct 的 reasoning → tool call → wait → reasoning 串行流程产生明显 idle latency；dLLM 可以任意顺序生成 | 优先解码 tool call，让 search API 在模型继续 reasoning 时并行执行 | ★★★★★ |

### DLLM-Searcher

- PDF: `02_search_agents_tool_use/SIGIR2026_DLLM_Searcher.pdf`
- Source: https://arxiv.org/abs/2602.07035
- SIGIR 2026 DOI: `10.1145/3805712.3809643`
- **dLLM-specific property**: flexible / any-order decoding 可改变 Agent 的执行调度，而不只是生成速度。
- P-ReAct 的核心：优先产生 tool-call region，外部检索开始执行后，模型继续生成 think region。
- **后续现象问题**：
  - tool call 到底提前多少才是最优？过早调用是否 query immature？
  - intermediate reasoning 的成熟度能否预测 search query quality？
  - latency-optimal timing 与 accuracy-optimal timing 是否存在系统冲突？

---

## C. Diffusion as Retriever / Retrieval Representation

| Paper | Venue | dLLM / Diffusion 特性 | Retrieval 价值 | 推荐度 |
|---|---|---|---|---:|
| **Diffusion-Pretrained Dense and Contextual Embeddings** | **ACL 2026 Industry Track** | diffusion pretraining 提供 bidirectional context | mean pooling / contextual embeddings 更适合长文档与 web-scale retrieval | ★★★★☆ |
| **DiffRetriever: Parallel Representative Tokens for Retrieval with Diffusion Language Models** | arXiv 2026 | 多个 `[MASK]` 位置可在一个 bidirectional pass 并行产生 representations | 低成本 multi-representative / ColBERT-style retrieval；存在 per-query adaptive budget 空间 | ★★★★★ |
| **Efficient, Property-Aligned Fan-Out Retrieval via RL-Compiled Diffusion (R4T)** | **ICML 2026** | diffusion 可以一次建模 set-valued outputs | 高效生成互补 retrieval embeddings，优化 diversity / coverage / complementarity | ★★★★☆ |

### Diffusion-Pretrained Dense and Contextual Embeddings

- PDF: `03_diffusion_as_retriever/ACL2026_Diffusion_Pretrained_Embeddings.pdf`
- Official: https://aclanthology.org/2026.acl-industry.69/
- 重点不是 dynamic RAG，而是证明 diffusion-pretrained backbone 的 **bidirectional context representation** 可以直接带来检索优势。
- 对 idea 的意义：dLLM 的中间 hidden state 不只是 generation state，也可能是 query / passage representation space。

### DiffRetriever

- PDF: `03_diffusion_as_retriever/ARXIV2026_DiffRetriever.pdf`
- Source: https://arxiv.org/abs/2605.07210
- Code: https://github.com/ielab/diffretriever
- **Phenomenon first**: AR multi-representative generation 的瓶颈在 sequential generation；dLLM 可一次预测 K 个 masked representative positions。
- 论文报告 per-query oracle 在固定 base model 上仍明显优于固定预算方案，直接留下：**adaptive representation budget**。
- 后续可观察 `query difficulty / semantic multiplicity / uncertainty → optimal K`。

### R4T

- PDF: `03_diffusion_as_retriever/ICML2026_R4T_RL_Compiled_Diffusion.pdf`
- Source: https://arxiv.org/abs/2603.06397
- Official: Google Research / ICML 2026
- 关键问题：很多 retrieval 目标是 set-level，而不是单个 document relevance。
- diffusion 的角色很自然：一次并行生成一组 retrieval embeddings，而不是自回归生成多个 subquery。
- 注意：主实验是 fashion/music set retrieval，不是经典 QA RAG；更适合作为“set-valued retrieval”方法论参考。

---

## D. dLLM Dynamics — 为“从现象找 Retrieval Idea”服务

这些论文并非都做外部检索，但它们暴露了最值得迁移到 dynamic retrieval 的 dLLM 特殊信号。

| Paper | Venue | dLLM 现象 | 可转成什么 retrieval signal？ | 推荐度 |
|---|---|---|---|---:|
| **Diffusion Language Models Know the Answer Before Decoding (Prophet)** | **ICLR 2026, Oral** | early answer convergence | retrieval need 是否也会提前收敛？何时应停止检索？ | ★★★★★ |
| **Lookahead Unmasking Elicits Accurate Decoding (LookUM)** | **ICML 2026** | greedy confidence unmasking 会走入错误 trajectory；错误路径提高 sequence uncertainty | trajectory uncertainty 可能优于 token entropy 作为 retrieval trigger | ★★★★★ |
| **Search or Accelerate (SOAR)** | **ICML 2026** | 低置信状态值得 search，高置信状态适合 parallel decode | 把 `search/accelerate` 改为 `retrieve/reason/parallelize` 的控制问题 | ★★★★★ |
| **DAPD** | **ICML 2026** | token-wise confidence 忽略 token dependency | dependency-aware knowledge deficit / localized retrieval | ★★★★★ |
| **Residual Context Diffusion (RCD)** | **ICML 2026** | discarded predictions retain useful contextual information | 用 residual / tentative state 构造更稳的 retrieval query | ★★★★★ |
| **Locally Coherent Parallel Decoding (CoDiLA)** | **ICML 2026** | 并行预测忽略 joint dependency 会破坏局部结构 | 哪些 retrieval facts / spans 需要 joint consistency？ | ★★★☆☆ |
| **d2: Trajectory Likelihood Estimation** | **ICML 2026** | diffusion reasoning 应在 trajectory level 评估，不应只看 terminal sample | trajectory likelihood / likelihood change 作为 retrieval utility signal | ★★★★☆ |
| **PoE-Bridge** | **ICML 2026** | DLM 与 AR distribution gap 很大；中间 bridge distribution 可提高 verification efficiency | 外部 evidence 是否可形成 retrieval-conditioned bridge / verifier | ★★★☆☆ |

### Prophet — Early Answer Convergence

- PDF: `04_dllm_dynamics_for_idea_mining/ICLR2026_Prophet_Early_Answer_Convergence.pdf`
- Source: https://arxiv.org/abs/2508.19982
- OpenReview: https://openreview.net/forum?id=s8bHSmuvwC
- **最值得模仿的 paper story**：作者不是先做 acceleration，而是先发现“答案在最终 decoding 前很久就已经内部确定”。
- 报告在 GSM8K / MMLU 上大量样本在约 half refinement steps 时就能得到正确答案。
- 方法 Prophet 只是根据 top-2 confidence gap 判断何时 all-in decode。
- Retrieval 迁移问题：**knowledge need 是否也存在 early convergence？**

### LookUM — Trajectory Error Accumulation

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_LookUM.pdf`
- Source: https://arxiv.org/abs/2511.05563
- Code: https://github.com/krafton-ai/LookUM
- **现象**：一次错误的 unmasking choice 会通过后续 trajectory 放大；错误轨迹的 sequence-level uncertainty 显著增加。
- Retrieval 迁移：不要只用某一步 entropy 触发检索，尝试测 **temporal / trajectory instability** 与真实 retrieval gain 的关系。

### SOAR — Confidence-Switched Search

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_SOAR.pdf`
- Source: https://arxiv.org/abs/2602.10953
- Code: https://github.com/duterscmy/SOAR
- **现象**：统一 decoding policy 不合理；低置信时应扩大 search，高置信时可 aggressive parallelization。
- Retrieval 迁移：可能存在 `retrieve / search trajectory / parallel decode` 的 state-dependent switching boundary。

### DAPD — Dependency-Aware Decoding

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_DAPD.pdf`
- Source: https://arxiv.org/abs/2603.12996
- Code: https://github.com/quasar529/DAPD
- **现象**：token marginals 并不能告诉你哪些 masked positions 可以安全同时更新；attention dependency graph 更关键。
- Retrieval 迁移：knowledge uncertainty 也可能是结构化的——不是“整条回答不知道”，而是若干彼此依赖的 semantic slots 缺知识。

### Residual Context Diffusion

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_Residual_Context_Diffusion.pdf`
- Source: https://arxiv.org/abs/2601.22954
- Code: https://github.com/yuezhouhu/residual-context-diffusion
- **现象**：remasking 时被丢弃的 token distributions 仍含后续 reasoning 所需上下文。
- 与 SARDI 是极强的互证：一个把 discarded signal 用于 **internal denoising**，一个把 discarded signal 用于 **external retrieval**。

### CoDiLA

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_CoDiLA.pdf`
- Source: https://arxiv.org/abs/2603.20216
- **现象**：并行采样 token marginals 不等于对 joint token structure 建模，代码等强局部依赖任务会出现 coherence artifacts。

### d2

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_d2_Trajectory_Likelihood.pdf`
- Source: https://arxiv.org/abs/2509.21474
- **现象/方法论**：masked diffusion 的 policy / reasoning 质量需要 trajectory likelihood，而 terminal output probability 不足以描述采样过程。
- Retrieval 迁移：可研究 `retrieval action → trajectory likelihood / stability change`，而不仅看最终 exact match。

### PoE-Bridge

- PDF: `04_dllm_dynamics_for_idea_mining/ICML2026_PoE_Bridge.pdf`
- Source: https://arxiv.org/abs/2606.08048
- Code: https://github.com/juntongshi48/poe-bridge
- **现象**：DLM proposal 与 AR target distribution 差距太大，使直接 importance sampling 成本高；桥接分布可以渐进修正。

---

## E. 2026 高价值预印本：只为挖现象，不冒充顶会

| Paper | Status | 现象 | 为什么值得看 |
|---|---|---|---|
| **Language Diffusion Models are Associative Memories Capable of Retrieving Unseen Data** | arXiv 2026 | UDDM 呈现 associative-memory-like basins；conditional entropy 可检测 memorization→generalization transition | 提供“conditional entropy 到底在 dLLM 中代表什么”的更基础视角 |
| **Introspective Diffusion Language Models** | arXiv 2026 | DLM 对自己早先生成 token 的接受并不一致，存在 introspective inconsistency | revision / self-acceptance 可能比单步 confidence 更适合度量知识冲突 |

### Associative Memory DLM

- PDF: `05_high_value_preprints/ARXIV2026_Associative_Memory_DLM.pdf`
- Source: https://arxiv.org/abs/2604.26841
- 最值得记录的是：conditional entropy 既可能反映 uncertainty，也可能反映模型所处的 memorization / generalization regime。
- 对 dynamic retrieval 的警告：**高 entropy ≠ 一定缺外部知识**。

### Introspective DLM

- PDF: `05_high_value_preprints/ARXIV2026_Introspective_DLM.pdf`
- Source: https://arxiv.org/abs/2604.11035
- 现象：DLM 经常不接受自己之前生成的 token；作者提出 introspective acceptance rate。
- Retrieval 迁移：可测试 `self-rejection / repeated revision` 是否比 entropy 更能预测 retrieval gain。

---

## F. Adjacent Dynamic Retrieval — 用来做 control / baseline / 对照现象

| Paper | Venue | 普通 AR-RAG 现象 | 对 dLLM 研究的作用 |
|---|---|---|---|
| **Adaptive Retrieval for Reasoning** | **ACL 2026 Long** | reasoning 中 retrieval timing 应自适应 | 用作“普通 LLM 如何做 when-to-retrieve”的对照 |
| **QuCo-RAG: Quantifying Uncertainty from the Pre-training Corpus for Dynamic RAG** | ACL Findings 2026 | logits / entropy 可能因 calibration 差而不能可靠预测 retrieval need | 非常适合与 dLLM trajectory signals 做 controlled comparison |

### Adaptive Retrieval for Reasoning

- PDF: `06_adjacent_dynamic_retrieval/ACL2026_Adaptive_Retrieval_for_Reasoning.pdf`
- Official: https://aclanthology.org/2026.acl-long.1734/

### QuCo-RAG

- PDF: `06_adjacent_dynamic_retrieval/ACLFindings2026_QuCo_RAG.pdf`
- Official: https://aclanthology.org/2026.findings-acl.812/
- 关键观点：普通 LLM 的 model-internal confidence / entropy 由于 calibration 问题并不天然可靠。
- 对本方向非常重要：如果 dLLM dynamic retrieval 仍然只用 instantaneous entropy，那可能没有真正利用 diffusion 的特殊结构。

---

# 最推荐优先精读的 8 篇

1. **ICLR 2026 — Diffusion Language Models Know the Answer Before Decoding**：学习“先发现现象再做方法”的范式。
2. **ICML 2026 — SARDI**：当前最直接的 dLLM × Dynamic Retrieval 核心论文。
3. **ICML 2026 — Residual Context Diffusion**：和 SARDI 共同证明 discarded signal 有价值。
4. **ICML 2026 — LookUM**：trajectory uncertainty / error propagation。
5. **ICML 2026 — DAPD**：position dependency 而非单点 confidence。
6. **SIGIR 2026 — DLLM-Searcher**：dLLM flexible order 如何改变 search-agent execution。
7. **SPREAD**：RAG 条件下 diffusion-specific semantic drift。
8. **ARAM**：retrieval-prior conflict 与 denoising-state-aware guidance。

读完这 8 篇后，再看 DiffRetriever / SOAR / d2 / R4T。
