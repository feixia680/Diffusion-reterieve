# 摘要（中文翻译）

Masked Diffusion Models 已成为有前景的非自回归序列生成范式，但其性能对 decoding strategy 高度敏感。本文发现主流 uncertainty-based decoding 会引入两类系统偏差：**rigid boundary bias** 与 **trivial token bias**，限制模型推理能力并降低生成质量。作者提出 UNCODE（UNmasking Calibration for DecOding DEbiasing），通过两个互补先验校准 uncertainty-based decoding，一方面塑造全局 decoding trajectory，另一方面促进更有内容的信息 token 被优先处理。三个先进 MDM、七个 reasoning/planning-intensive benchmark 上，UNCODE 相比已有 decoding strategy 稳定提升超过 7%，并达到与同规模 AR 模型相近的表现。

# 1. TL;DR

**uncertainty 不是中性观测量，它会被 decoding mechanism 系统性扭曲。**

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.311/
- Code：https://github.com/NEUIR/Uncode
- 标签：`Decoding bias`、`Uncertainty calibration`

# 3. 背景

大量 DLM 解码器按 uncertainty/confidence 决定 unmask 顺序。

# 4. Existing Assumption

> uncertainty ranking 直接反映位置的重要性/可靠性。

# 5. 核心现象

uncertainty heuristic 会偏向固定边界和 trivial/easy token，造成全局 trajectory bias。

# 6. Why

低 entropy token 往往是标点、格式、简单词，并不一定是对 reasoning 最关键的信息；空间/位置结构也会影响 confidence。

# 7. Formulation

在原 uncertainty score 上加入 global trajectory prior 与 informativeness prior 进行 calibration。

# 8. 方法

UNCODE 重新标定 unmask score，避免只追求“最容易确定的 token”。

# 9. 关键理解

**easy-to-predict ≠ important-to-decide**。

# 10. 流程

base uncertainty → debias priors → calibrated score → select positions。

# 11. 实验

3 个 MDM × 7 个 reasoning/planning benchmark。

# 12. 结果

平均/持续 >7% 提升，并接近同尺度 AR。

# 13. Ablation

两类 bias prior、不同 decoding base、不同任务。

# 14. Failure

校准先验仍可能与特定 domain 不匹配。

# 15. Limitations

研究的是 decoding quality，不直接验证 knowledge uncertainty。

# 16. Retrieval 关系

这是我们最重要的警告之一：若直接用 entropy 触发 retrieval，可能只是在响应 decoding bias，而不是真实 knowledge gap。

# 17. Idea Mining

构造 `raw entropy`、`debiased entropy`、`revision`、`entity stability` 对 retrieval gain 的预测比较。

# 18. Pilot + Figure

按 token 类型分组（entity/relation/number/function word），画 entropy 与 retrieval gain 的散点/相关性；验证 trivial token bias 是否污染 retrieval trigger。

# 19. 复现价值

★★★★★；training-free decoding，pilot 非常合适。

# 20. 记住什么

1. uncertainty signal 本身需要校准。
2. 我们找 idea 时必须优先找“反直觉 mismatch”。
