# 摘要（中文翻译）

Masked Diffusion Models 通过迭代 unmask token 生成文本，其性能高度依赖推理时的 unmask 顺序。常见 confidence-based sampling 很短视：只优化局部决策，无法利用额外 test-time compute，而且早期 decoding 错误会级联。LookUM 将 sampling 重写为所有可能 unmasking order 上的路径选择，不需要外部 reward model。它用 path generator 从 unmask set pool 中提出候选路径，再用 verifier 计算路径不确定性并通过 importance sampling 选择。实验发现错误 unmasking 会显著抬高 sequence-level uncertainty，因此该信号可以用于避开错误轨迹。六个数学、规划、代码 benchmark 上均有一致提升，只需 2–3 条路径即可达到峰值表现。

# 1. TL;DR

**瞬时 confidence 不是 trajectory quality。** 一次早期错误会级联，而 sequence-level uncertainty 能暴露错误路径。

# 2. 基本信息

- 题目：Lookahead Unmasking Elicits Accurate Decoding in Diffusion Language Models
- ICML 2026
- Source：https://arxiv.org/abs/2511.05563
- Code：https://github.com/krafton-ai/LookUM

# 3. 背景

greedy confidence decoding 每步只看当前最有把握的位置。

# 4. Existing Assumption

> 当前最 confident 的局部动作也是全局最优动作。

# 5. 核心现象

错误 unmasking 会在后续累积，并显著增加整条 sequence 的 uncertainty。

# 6. Why

早期 token 一旦固定，其他位置会以它为上下文继续推断；局部错误改变整个 conditional distribution。

# 7. Problem

从单步 token selection 升级为 trajectory/path selection。

# 8. 方法

提出少量候选 unmask paths，用内部 uncertainty verifier 评估，再 importance sample 选择。

# 9. 关键量

不是单 token entropy，而是候选 trajectory 的 sequence-level uncertainty。

# 10. 流程

state → candidate paths → lookahead rollout → uncertainty verification → select path。

# 11. 实验

六类 reasoning/planning/coding benchmark；LLaDA 与 LLaDA1.5。

# 12. 主结果

2–3 paths 已足够；base LLaDA+LookUM 可接近 RL-tuned LLaDA1.5，且还能继续提升后者。

# 13. Ablation

path 数、verifier、lookahead depth 是关键。

# 14. Failure Case

若模型所有候选路径都共享同一个错误 prior，低 uncertainty 仍可能错误。

# 15. Limitations

额外 rollout 带计算成本；uncertainty 并非 factual uncertainty。

# 16. Retrieval 关系

非常适合挑战“entropy 高就检索”：**temporal trajectory instability 可能比 instantaneous entropy 更能预测 retrieval gain**。

# 17. Idea Mining

对同一样本同时计算 token entropy、revision rate、trajectory uncertainty，和 intervention-based retrieval gain 做相关/分类比较。

# 18. Motivation Figure

三条 ROC/AUC：Entropy vs Revision vs Trajectory Uncertainty 对“retrieval 是否带来正增益”的预测能力。

# 19. 复现价值

★★★★★；无需训练；是第一批 pilot 强烈推荐 baseline。

# 20. 记住什么

1. early mistake 会级联。
2. trajectory signal 比单步 signal 更值得研究。
