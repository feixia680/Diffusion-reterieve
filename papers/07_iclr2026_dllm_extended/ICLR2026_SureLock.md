# 摘要（中文翻译）

Masked DLM 通过迭代 sampling 逐渐 unmask token，但即使许多已 unmask token 实际已经固定，模型仍在每一步为所有位置重复 attention 与 FFN 计算，造成大量浪费。SureLock 在某个位置的 posterior 跨 steps 稳定后满足 “sure” 条件，就锁住该位置：后续跳过其 query projection 和 FFN，同时缓存 K/V 供其他位置继续 attention。于是主计算复杂度从 `O(N²d)` 降为 `O(MNd)`，其中 M 是仍未锁定位置数并随迭代下降。在 LLaDA-8B 上，SureLock 相比同 sampler 可减少约 30–50% algorithmic FLOPs 且保持相近生成质量；理论上，lock 时刻的 local KL 足以约束最终 token probability 偏差。

# 1. TL;DR

一个 token 是否“稳定”可以通过跨 step posterior 直接检测，而且稳定后真的可以停止对它投入计算。

# 2. 基本信息

- ICLR 2026
- 作者：Daisuke Oba 等
- 标签：`Posterior stability`、`Early lock`

# 3. 背景

所有位置每步等价计算极浪费。

# 4. Assumption

> 已 unmask token 仍可能随时变化，因此必须继续完整更新。

# 5. 现象

许多 token posterior 很早稳定。

# 6. Why

一旦上下文核心语义成形，部分位置的 conditional distribution 变化趋近于零。

# 7. Formulation

跨步 posterior divergence / local KL 小于阈值 → lock。

# 8. 方法

skip locked token 的 query/FFN；缓存其 K/V。

# 9. 关键理解

SureLock 提供比单步 confidence 更强的 **temporal stability criterion**。

# 10. 流程

predict → compare posterior history → lock stable positions → continue only M positions。

# 11. 实验

LLaDA-8B；FLOPs 与 generation quality。

# 12. 结果

30–50% FLOPs 节省，质量基本不变。

# 13. Ablation

稳定窗口、阈值、不同 token 类别/步数。

# 14. Failure

在有外部 evidence intervention 时，原本稳定 token 可能需要重新打开；static lock 可能阻碍知识纠正。

# 15. Limitations

优化 compute，不判断 stable token 是否 factually correct。

# 16. Retrieval 关系

**stable ≠ correct** 是关键 research question。若 stable-wrong 样本普遍存在，它们正是 confidence-based retrieval 会漏掉的案例。

# 17. Idea Mining

研究 posterior stability 对 retrieval gain 是负相关还是存在 U 型/双峰关系。

# 18. Pilot

四组：stable-correct、stable-wrong、unstable-correct、unstable-wrong，分别测 retrieval gain。

# 19. 复现

★★★★★；training-free；强 pilot source。

# 20. 记住什么

1. temporal stability 可直接测。
2. 下一步要问：稳定错误如何识别？
