# 摘要（中文翻译）

LLM 虽有强推理能力，但参数知识在时效性、准确性和完整性方面有限，导致事实幻觉。将 reasoning 与 RAG 结合又面临 task decomposition 低效与 redundant retrieval，后者会引入噪声并降低回答质量。DeepRAG 把 retrieval-augmented reasoning 建模为 MDP，使系统在逐步分解 query 的过程中动态决定：当前步骤是调用外部检索，还是依靠参数推理。实验显示 DeepRAG 提高 retrieval efficiency，并使回答准确率提升约 25.41%。

# 1. TL;DR

Retrieval Orchestration 的经典基线：**每个 reasoning step 做 `retrieve or reason` 决策**。

# 2. 基本信息

- ICLR 2026
- 标签：`Adaptive Retrieval`、`MDP`

# 3. 背景

固定每步检索会产生冗余和噪声；永不检索又会知识不足。

# 4. Assumption

> retrieval schedule 可以固定。

# 5. 现象

不同 reasoning steps 的 external knowledge need 不同。

# 6. Why

部分子问题可由 parametric knowledge 解，部分需要新事实；每次搜索还有 token/latency/noise cost。

# 7. Formulation

state = 当前 query/subproblem/reasoning；action = `{retrieve, reason}`；reward = answer quality/cost。

# 8. 方法

迭代 decomposition + MDP policy。

# 9. 关键理解

它把 RAG 从 pipeline 变成 sequential decision process。

# 10. 流程

question → decompose → choose retrieve/reason → update state → repeat → answer。

# 11. 实验

复杂 QA，评价 accuracy 与 retrieval count。

# 12. 结果

准确率约 +25.41%，同时减少无效检索。

# 13. Ablation

固定 retrieval、无 decomposition、不同 policy。

# 14. Failure

action space 太粗：不知道 which retriever、how query、top-k、stop。

# 15. Limitations

不是 dLLM；state 没有 diffusion-specific temporal signals。

# 16. dLLM 迁移

把 state 换成 `denoising step + tentative tokens + revision + stability + entropy`，就是我们的核心方向。

# 17. Idea Mining

验证 dLLM temporal state 是否能比普通 reasoning text 更准确预测 retrieve-vs-reason。

# 18. Pilot

用简单 logistic/threshold classifier 只输入不同 signal，比较 retrieval gain prediction AUC。

# 19. 复现

★★★★★；baseline 价值★★★★★。

# 20. 记住什么

DeepRAG 给 action formulation；dLLM papers 给更丰富 state。二者结合是很自然的研究路线。
