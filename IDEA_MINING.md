# Idea Mining: 从 dLLM 现象出发，而不是从模块拼接出发

本项目后续找 idea 的默认原则：

> **不要先问“能不能把 retrieval 加到 diffusion LLM”。先问“diffusion LLM 暴露了什么 AR LLM 没有的稳定现象，这个现象是否揭示 knowledge need / retrieval gain / retrieval failure？”**

---

# 1. 统一研究模板

每一个候选 idea 必须先填下面 8 个字段：

```text
1. Existing assumption
2. Observable variable
3. Pilot observation
4. Unexpected phenomenon
5. Why it happens
6. Retrieval implication
7. Minimal method
8. Key controlled experiment
```

如果 1–5 讲不清楚，不进入 method design。

---

# 2. 2026 论文已经暴露出的 dLLM 特殊现象

## Phenomenon A — Early Answer Convergence

来源：ICLR 2026 Prophet。

### 已知观察

最终 answer 尚未完整 decode 时，内部预测往往已经稳定到正确答案。

### 不要直接想的方法

`early retrieval router`。

### 应先问的问题

- `answer convergence` 与 `knowledge need convergence` 是否同步？
- 模型什么时候已经“知道自己需要哪个外部实体”？
- 如果答案已经稳定，再 retrieval 会不会反而引入 noise？
- early convergence 能否预测 **stop retrieving**？

### Pilot

对每个 query、每个 denoising step 记录：

```text
step
answer correctness / answer proxy
entity set
entity stability
entropy
revision count
retrieval gain if retrieval is injected at this step
```

然后画：

`retrieval gain vs. convergence step`。

---

## Phenomenon B — Discarded Tokens Carry Information

来源：ICML 2026 SARDI + Residual Context Diffusion。

### 已知观察

低置信、最终被 remask / 丢弃的预测并不是纯噪声：

- SARDI：可暴露未来 bridge entity；
- RCD：仍能作为后续 denoising 的 contextual residual。

### 核心新问题

不是“tentative token 能不能检索”——SARDI 已经回答了。

而应该进一步研究：

- 哪些 discarded tokens 是 **useful lookahead**，哪些是 hallucination？
- usefulness 是否可以从 temporal stability / attention / revision history 判断？
- discarded signal 的价值是否具有 **position / semantic-role specificity**？
- entity / relation / number 哪类 tentative token 最值得转成 retrieval query？

### Pilot

为每个 tentative token/span 记录：

```text
confidence
first-emergence step
survival length
revision frequency
semantic type
final correctness
retrieval hit gain
answer gain
```

目标先做 correlation / stratified analysis，不训练 router。

---

## Phenomenon C — Instantaneous Confidence Is Myopic

来源：ICML 2026 LookUM / SOAR；相邻对照：ACL Findings 2026 QuCo-RAG。

### 已知观察

- LookUM：局部 confidence 最优不代表 trajectory 最优；错误 unmasking 会提高后续 sequence uncertainty。
- SOAR：低置信与高置信状态应使用不同 decoding policy。
- QuCo-RAG：普通 LLM logits / entropy 因 calibration 不佳，未必能可靠反映 retrieval need。

### 最值得验证的问题

> **dLLM dynamic retrieval 中，temporal instability 是否比 instantaneous entropy 更能预测 retrieval gain？**

### 候选变量

- token entropy
- top1-top2 margin
- entropy slope
- revision count
- entity stability
- trajectory likelihood
- self-acceptance / introspective consistency
- attention dependency

### Key controlled experiment

固定：model / retriever / corpus / top-k / query。

只比较 trigger signal：

```text
Entropy
vs Confidence gap
vs Revision
vs Entity stability
vs Temporal instability
```

预测目标统一为：

`oracle retrieval gain = accuracy(with retrieval now) - accuracy(no retrieval now)`。

这比直接比较最终 QA accuracy 更能回答“哪个 signal 真正解释 need-to-retrieve”。

---

## Phenomenon D — Token Knowledge Deficit Is Structured, Not Global

来源：ICML 2026 DAPD。

### 已知观察

masked positions 之间存在明显 dependency；单 token marginal confidence 无法判断哪些位置能同时安全更新。

### Retrieval 问题

模型可能并不是“整个 answer 不知道”，而是：

```text
Entity: stable
Relation: unstable
Date: unstable
Syntax: stable
```

因此 retrieval query 可能应该只针对不稳定的 semantic slots。

### Pilot

把 masked positions 聚类成：

- entity
- relation
- attribute
- number/date
- function word / syntax

测试：

`position-wise instability → factual error type → retrieval gain`。

如果成立，可进一步研究 localized / slot-aware retrieval。

---

## Phenomenon E — Retrieval Causes Diffusion-Specific Semantic Drift

来源：SPREAD。

### 已知观察

DLM + RAG 后，response 可能在 iterative denoising 中逐步偏离 query 语义。

### 后续不要只做“更强 guidance”

先问：

- RSD 是 monotonic 还是 oscillatory？
- drift 发生在 evidence injection 后多久？
- correct evidence 与 noisy evidence 的 drift pattern 是否不同？
- drift 与 entity revision / relation revision 是否同步？
- 某些 timestep 是否是 **point of no return**？

### Pilot curve

横轴：denoising step。

纵轴同时画：

- query-response semantic similarity
- evidence-response similarity
- entity consistency
- answer correctness proxy
- entropy / revision

把正确与错误样本分开画。

---

## Phenomenon F — Retrieval-Prior Conflict Is Time-Dependent

来源：ARAM。

### 已知观察

retrieved context 与 model parametric prior 冲突时，固定 guidance scale 不合理。

### 更基础的问题

- model 什么时候开始形成稳定 parametric belief？
- evidence 如果在 belief 形成前进入 vs 形成后进入，效果是否不同？
- 纠正错误 belief 和破坏正确 belief 的 trajectory pattern 是否可区分？

### Pilot

人为构造三组 evidence：

```text
supportive
irrelevant
contradictory
```

在不同 denoising step 注入，得到：

`step × evidence quality × final accuracy / revision / semantic drift`。

先画 phase diagram，再设计 adaptive guidance。

---

## Phenomenon G — Retrieval Timing Can Exploit Parallelism

来源：SIGIR 2026 DLLM-Searcher。

### 已知观察

传统 ReAct：

```text
reason → tool call → wait → reason
```

存在串行 idle time。

dLLM flexible generation order 可以先完成 tool-call region，再继续 think。

### 后续现象

- search query quality 随 denoising step 怎么变化？
- query quality 是否“先快速成熟、后缓慢变化”？
- tool call 越早越好吗？
- early tool call 节省 latency，但是否增加 bad retrieval？

### Pilot

每个 timestep 导出 search query，测：

```text
Recall@k
answer utility
query semantic completeness
latency saved
```

画 Pareto：`retrieval quality vs. call time`。

---

## Phenomenon H — Diffusion Can Produce Multiple Retrieval Representations in Parallel

来源：DiffRetriever。

### 已知观察

多个 masked representative positions 可一次产生 K 个 retrieval representations；AR 模型的多 token representation 需要串行生成。

### 最重要的开放问题

论文已经提示 per-query oracle 优于 fixed K：

> **不同 query 的最佳 representation budget 不一样。**

### Pilot

对每个 query 计算：

```text
K = 1,2,4,8,16
NDCG / Recall
latency
query ambiguity
entity count
semantic diversity
uncertainty
```

寻找：`query property → optimal K`。

---

# 3. 我们真正想发现的“反直觉结果”

优先级高于普通正相关。

## 例 1

Existing assumption:

> entropy 高 → 模型不知道 → 应检索。

Possible observation:

> entropy 与 retrieval gain 相关很弱，但 cross-step revision instability 与 retrieval gain 强相关。

Why:

> entropy 是 instantaneous ambiguity；revision 是 temporal semantic disagreement。

这就能自然得到 trajectory-aware retrieval trigger。

## 例 2

Existing assumption:

> retrieval 越早越好，因为可更早提供知识。

Possible observation:

> 太早时 query semantics 尚未成熟，retrieval quality 低；中期最好；晚期又来不及纠正 trajectory。

得到一个倒 U 型 `retrieval timestep → gain` 曲线。

这会自然导向 state-aware retrieval timing。

## 例 3

Existing assumption:

> tentative tokens 越高 confidence 越值得用于 retrieval。

Possible observation:

> 中等 confidence、跨 step 稳定出现的 entity 比单步高 confidence entity 更能预测正确 bridge entity。

得到 `confidence ≠ reliability`，而 `temporal stability` 更关键。

---

# 4. 第一批推荐 Pilot Experiments

不要同时做十个。优先以下三个。

## Pilot 1 — Which signal predicts retrieval gain?

模型：LLaDA / Dream。

数据：multi-hop QA。

每一步记录：

- entropy
- top-2 margin
- revision
- entity stability
- tentative entity set

对每一步做一次 oracle retrieval intervention，测最终 answer gain。

目标：找到真正解释 `when to retrieve` 的 diffusion-specific signal。

**这是当前最值得优先做的 pilot。**

## Pilot 2 — Retrieval Timing Curve

固定同一 retrieval query / retriever / evidence budget，只改变 evidence 注入 timestep。

测试：

`early / 25% / 50% / 75% / late`。

画：

- final accuracy
- semantic drift
- entity stability
- revision rate

目标：验证是否存在 universal / query-dependent optimal retrieval window。

## Pilot 3 — Correct vs Hallucinated Tentative Entities

从每个 denoising trajectory 抽取 entity emergence history。

对比最终正确 / 错误 entity：

- emergence time
- survival duration
- revision count
- confidence trajectory
- attention pattern

目标：回答 SARDI 后最直接的问题：

> **如何区分 useful lookahead 与 hallucinated lookahead？**

---

# 5. 暂时不建议的 Idea 形式

以下 story 不够：

- “把 uncertainty 加进 SARDI”。
- “给 diffusion RAG 加一个 router”。
- “用 RL 学什么时候 retrieve”。
- “把普通 Adaptive RAG 迁移到 LLaDA”。
- “把三个 signal concat 后过 MLP”。

除非前面先有清晰、可复现的 observation 说明：**现有假设为什么错、diffusion dynamics 新暴露了什么信息。**

---

# 6. 最终论文 Story 模板

```text
Existing systems assume X.

By tracing the denoising trajectory, we observe Y,
a stable and previously overlooked phenomenon unique / especially visible in dLLMs.

Y reveals that X fails because Z.

We therefore formulate retrieval decision as ...

Motivated directly by this observation, we propose a minimal mechanism M.

Controlled experiments isolate Y and show that M improves both retrieval utility and final task quality.
```

这个仓库后续新增论文时，优先记录 **Observation / Why / Unexplained**，方法细节放第二优先级。
