# 摘要（中文翻译）

LLM 在复杂推理任务上能力很强，但自回归生成使探索多条推理路径的成本很高。DLM 则采用并行、非自回归生成机制，可以高效地产生多样候选输出。基于这种互补性，本文探索一种协同推理框架：让 DLM 高效生成多样 intermediate reasoning thoughts，再让 AR LLM 作为 evaluator，根据合理性与正确性评估和选择候选。该设计把 proposal generation 与 evaluation 解耦，分别利用 diffusion 的高效探索能力与 AR 的因果评估能力，与认知心理学中的 divergent-convergent thinking 相呼应。多个数学与逻辑推理 benchmark 上，框架在保持竞争性甚至更高准确率的同时提升推理效率。

# 1. TL;DR

DLM 很适合作为 **parallel thought proposer**：一次产生多个候选思路，再让更强 evaluator 收敛选择。

# 2. 基本信息

- 题目：Diffuse Thinking: Exploring Diffusion Language Models as Efficient Thought Proposers for Reasoning
- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.1231/
- 标签：`Parallel reasoning`、`Proposal-Evaluation`

# 3. 背景

Tree-of-Thought/Best-of-N 依赖多次 AR rollout，成本高。

# 4. Existing Assumption

> 多条 reasoning path 必须通过多次顺序生成得到。

# 5. 核心现象

DLM 的并行/any-order generation 能用较低额外延迟产生多样 thought candidates。

# 6. Why

masked positions 可同时承载不同潜在推理内容，不必像 AR 一条路径完成后再开下一条。

# 7. Problem Formulation

proposal model 负责覆盖 candidate thought space，evaluator 负责选择高 utility candidate。

# 8. 方法总览

DLM proposer → diverse intermediate thoughts → AR evaluator → select → continue/final reasoning。

# 9. 方法理解

这是典型“探索与评估解耦”。真正的价值在于 DLM 提供 cheap diversity，而不是让它独自承担最终 verifier。

# 10. 流程

输入问题 → DLM 并行提出 thoughts → evaluator 打分 → 选优 → 组合/继续推理。

# 11. 实验

数学与逻辑 reasoning；比较效率与准确率。

# 12. 主结果

在提升候选探索效率的同时保持或超过 baseline reasoning accuracy。

# 13. Ablation

candidate 数量、DLM proposer、AR evaluator、selection rule。

# 14. Failure Case

候选多样不等于有用；如果所有 thoughts 都共享同一个事实缺口，内部扩散探索无法替代 external retrieval。

# 15. Limitations

需要两个模型；evaluator 成本仍存在；主要验证 reasoning，不是检索。

# 16. 与 Diffusion Retrieval 的关系

可自然迁移成 `multiple tentative thoughts → multiple retrieval queries`，让 dLLM 并行探索不同 search intent。

# 17. Idea Mining

关键问题：多个 tentative query 是互补还是重复？dLLM 是否比 AR sequential query expansion 产生更高 evidence coverage？

# 18. Pilot + Motivation Figure

一次生成 K 个 tentative query，比较 DLM parallel vs AR sequential 的 evidence coverage、redundancy、latency。画 coverage-latency Pareto curve。

# 19. 复现价值

2×A100：★★★★☆；idea source：★★★★★。

# 20. 记住什么

1. dLLM 的并行优势可以用于“搜索意图探索”。
2. 内部 thought diversity 与外部 evidence diversity 可以统一研究。
