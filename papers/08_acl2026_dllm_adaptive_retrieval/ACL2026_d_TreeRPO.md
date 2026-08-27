# 摘要（中文翻译）

RL 对提升 dLLM reasoning 很重要，但现有 policy optimization 有两个可靠性瓶颈：其一，reward 稀疏，粗粒度或不可验证信号使 advantage 难准确计算；其二，常用 probability estimate 没有考虑所有 decoding orders 上无偏期望与单次 forward 估计之间的差距。d-TreeRPO 使用 tree-structured rollouts 与自底向上的 advantage computation，在可验证 outcome reward 基础上产生更细粒度 step-wise signal；理论上作者证明提高 prediction confidence 能缩小无偏期望概率与单步估计之间的 gap，并据此加入 time-scheduled self-distillation，在训练后期增强 confidence。多个 reasoning benchmark 上，d-TreeRPO 明显优于 baseline，相比 base model 在 Sudoku、Countdown、GSM8K、Math500 上分别取得显著提升。

# 1. TL;DR

DLM 的 **多 decoding order** 会让 policy probability / credit assignment 变得不可靠，必须显式处理 trajectory tree。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.994/
- 标签：`Tree rollout`、`Policy optimization`

# 3. 背景

传统 RL 习惯把一条采样轨迹当成 policy 的代表。

# 4. Assumption

> 单次 decoding order 的 probability estimate 足够稳定。

# 5. 核心现象

多 order 下存在期望概率偏差，reward 又稀疏，二者叠加让 advantage 噪声很大。

# 6. Why

同一输出可以由不同 unmask order 到达；单路径无法代表所有可能顺序。

# 7. Formulation

tree rollout 枚举/共享多条分支，自底向上聚合可验证 reward。

# 8. 方法

d-TreeRPO + scheduled self-distillation。

# 9. 关键理解

trajectory branching 不只是搜索技巧，也是估计 uncertainty/utility 的工具。

# 10. 流程

branch rollout → leaf reward → bottom-up advantages → policy update。

# 11. 实验

Sudoku、Countdown、GSM8K、Math500 等。

# 12. 结果

相比 base model：Sudoku +86.2%、Countdown +51.6%、GSM8K +4.5%、Math500 +5.3%（论文报告口径）。

# 13. Ablation

tree structure、self-distill schedule、confidence effect。

# 14. Failure

rollout tree 计算成本高；branching policy 仍可能共享同一错误 prior。

# 15. Limitations

RL 训练较重，不适合 2×A100 直接主攻。

# 16. Retrieval 关系

可以借它的 **branching controlled experiment**：在同一 intermediate state 分支成 retrieve/no-retrieve，再直接估计 retrieval advantage。

# 17. Idea Mining

无需训练 RL，先用 paired branch rollout 得到 ground-truth retrieval gain label。

# 18. Pilot

每个 state 克隆两条 trajectory：A 不检索，B 检索；最终 score 差就是局部 retrieval advantage。

# 19. 复现价值

完整 RL：★★☆☆☆；paired intervention 思想：★★★★★。

# 20. 记住什么

找 retrieval signal 前，先用分支实验建立可信的“真实 gain”标签。
