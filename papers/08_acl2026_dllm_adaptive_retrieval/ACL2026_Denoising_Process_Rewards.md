# 摘要（中文翻译）

Diffusion-based LLM 为文本生成提供非自回归替代方案，但复杂推理仍然困难。RL 已成为提升这类模型的重要后训练方法，但现有方法主要依赖 outcome reward，只在最终结果上给监督，无法直接评价 denoising process，因而可能得到结构混乱、难解释且对最终预测支持不稳定的推理过程。本文提出 denoising process reward：在 diffusion trajectory 上定义 process-level reinforcement signal，通过估计不同中间 denoising interval 对最终任务结果的贡献，鼓励模型选择持续推动正确答案形成的轨迹。作者还提出可复用标准 rollout 的随机估计器，使过程监督具备实际训练可行性。多个困难 reasoning benchmark 上，该方法提升了 reasoning stability、interpretability 与最终表现。

# 1. TL;DR

只给 final reward 太粗；**不同 denoising interval 对最终正确性的贡献可以被单独估计**。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.1978/
- 标签：`Process Reward`、`Trajectory Attribution`

# 3. 背景

现有 RLVR 通常 final correct=1 / wrong=0。

# 4. Assumption

> 最终 outcome reward 足够训练整个 diffusion trajectory。

# 5. 核心现象

同样最终答案背后，不同 denoising interval 对正确推理贡献不同；粗粒度 outcome reward 无法定位“哪一步真正有用”。

# 6. Why

DLM 的 trajectory 包含多次并行 revision，错误/正确 signal 被终点 reward 混在一起。

# 7. Problem Formulation

估计 interval `I_t` 对 final task reward 的边际贡献，得到 process reward `r_t`。

# 8. 方法

基于 rollout attribution 构造 denoising process reward，并设计 stochastic estimator 复用已有 rollout。

# 9. 关键理解

它把“trajectory 中每一步值不值得”变成可学习/可估计量。

# 10. 流程

rollout → interval contribution estimation → process reward → RL update。

# 11. 实验

复杂 reasoning benchmark；评价最终性能、稳定性与可解释性。

# 12. 主结果

process-level reward 相比 outcome-only training 持续提升 reasoning quality。

# 13. Ablation

process reward、不同 interval granularity、estimator reuse。

# 14. Failure

贡献估计仍是近似，可能受后续 trajectory interaction 影响。

# 15. Limitations

训练成本较高；reward attribution 与 causal contribution 未必完全等价。

# 16. Retrieval 关系

可把相同思想迁移成 **Retrieval Process Reward**：某 timestep 的 retrieval 到底给 final answer 带来多少边际贡献？

# 17. Idea Mining

定义 `G_t = score(retrieve at t)-score(no retrieve)`，把 retrieval intervention 当 process-level attribution。

# 18. Pilot

同一批问题枚举 retrieval timestep，得到 gain curve；再训练/测试 signal 是否能预测高 gain interval。

# 19. 复现价值

完整 RL：★★★☆☆；只做 retrieval attribution pilot：★★★★★。

# 20. 记住什么

1. diffusion trajectory 应逐段归因。
2. 我们的 retrieval timing 研究也应从“哪一步真正有增益”出发。
