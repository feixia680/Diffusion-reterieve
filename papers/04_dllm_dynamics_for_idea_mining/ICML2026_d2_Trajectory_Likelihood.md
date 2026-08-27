# 摘要（中文翻译）

DLM 已在文本生成中取得有竞争力表现，但用 RL 提升推理能力仍是活跃问题。本文提出面向 masked DLM 的 d² reasoning framework，其核心是新的 policy-gradient 算法，需要准确估计 sampling trajectory likelihood。由于 masked DLM 直接计算 trajectory likelihood 代价很高，作者针对不同模型类别提出不同 estimator。对于支持 any-order decoding 的 DLM，d²-AnyOrder 可在一次模型 pass 中得到精确 trajectory likelihood；作者同时发现 any-order 并非所有常用 DLM 在实践中都真正支持。对标准 masked diffusion，d²-StepMerge 用可解析方式近似 trajectory likelihood，在计算与近似精度之间折中。d² 在多种 DLM 上明显优于常用 RL baseline，并在 Countdown、Sudoku、GSM8K、MATH500 等推理任务上取得强结果。

# 1. TL;DR

评估 dLLM policy 不能只看最终输出概率，**整个 denoising trajectory 的概率结构很重要**。

# 2. 基本信息

- ICML 2026
- Source：https://arxiv.org/abs/2509.21474
- 标签：`Trajectory likelihood`、`RL`

# 3. 背景

AR policy likelihood 是 token logprob 之和；dLLM 存在多种 decoding order，同一终点可由很多路径到达。

# 4. Assumption

> 用单步或终点 probability 可以近似 diffusion policy probability。

# 5. 现象

不同 trajectory 到同一答案并不等价；any-order 支持也不是“理论上有”就“工程上真的有”。

# 6. Why

masked diffusion 的 transition 与 ordering 都是随机变量，policy gradient 需要正确的 path probability。

# 7. Formulation

目标是估计 `p(trajectory)` 而不仅是 `p(final x)`。

# 8. 方法

AnyOrder exact estimator + StepMerge approximate estimator。

# 9. 关键理解

StepMerge 用合并多个细粒度 transition 降低计算，牺牲部分精度；这提供可调 compute-quality tradeoff。

# 10. 流程

rollout → trajectory likelihood estimation → advantage / policy gradient update。

# 11. 实验

逻辑与数学 reasoning；比较 RL baselines。

# 12. 结果

d² 提升多个任务，并建立更可靠 diffusion RL。

# 13. Ablation

estimator 精度、merge 程度、模型是否真正 any-order。

# 14. Failure

近似 likelihood 偏差可能直接污染 RL gradient。

# 15. Limitations

训练开销较大；与 external retrieval 不是直接问题。

# 16. Retrieval 关系

可以定义 `retrieval action → trajectory likelihood/stability change`，不再只用最终 EM 评价 retrieval utility。

# 17. Idea Mining

如果 evidence 是正确的，是否会让后续正确 trajectory 的 likelihood 更集中？错误 evidence 是否相反？

# 18. Pilot

固定问题，在不同 timestep 注入 evidence，比较注入前后的 trajectory likelihood、revision 与 final gain。

# 19. 复现

★★★☆☆；RL 成本中高；idea source ★★★★☆。

# 20. 记住什么

trajectory 是 dLLM 的一等公民；Retrieval Orchestration 也应做 trajectory-level evaluation。
