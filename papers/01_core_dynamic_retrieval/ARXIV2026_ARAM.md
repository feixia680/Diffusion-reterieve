# 摘要（中文翻译）

RAG 通过引入外部知识提升事实性，但当检索上下文噪声较大、不可靠或与模型参数知识冲突时，会产生 retrieval-prior conflict 并降低生成质量。该问题在自回归模型中已有研究，但在扩散语言模型中仍缺乏探索，而扩散模型的迭代去噪又为证据整合带来新的挑战。本文提出 ARAM（Adaptive Retrieval-Augmented Masked Diffusion），一种无需训练的自适应 guidance 框架。ARAM 根据检索上下文引起的分布偏移的信噪比（SNR），在去噪过程中动态校准 guidance scale：当检索证据提供可靠纠正信息时增强 guidance；当信号噪声大或缺乏支持时抑制 guidance。多个知识密集型 QA 基准上的实验表明，ARAM 相比竞争性 RAG 基线能提升整体问答性能。

# 1. TL;DR

**retrieved context 越强不一定越好。** ARAM 把“是否相信外部 evidence”变成一个随 denoising state 动态变化的问题。

# 2. 基本信息

- 题目：Adaptive Guidance for Retrieval-Augmented Masked Diffusion Models
- 作者：Jaemin Kim, Jong Chul Ye
- 状态：arXiv 2026
- Source：https://arxiv.org/abs/2603.17677
- 标签：`Retrieval-prior conflict`、`Adaptive guidance`、`Training-free`

# 3. 背景

普通 RAG 往往默认固定方式拼接 context，或使用固定强度的 context guidance。但 evidence 质量对不同 query、不同时间步都可能不同。

# 4. Existing Assumption

> retrieved context 应始终以相同强度影响生成。

# 5. 核心现象

- noisy / contradictory evidence 会伤害 dLLM。
- external context 引起的分布变化有时是“有用纠正”，有时只是“噪声扰动”。
- 同一份 evidence 在不同 denoising state 下的价值并不恒定。

# 6. 为什么会出现？

早期状态高度不确定，外部证据可能提供方向；但如果证据本身错误，也可能造成强 priming。晚期状态已经形成较强 parametric prior，外部 context 与 prior 的冲突模式又不同。因此 guidance strength 应依赖状态。

# 7. Problem Formulation

比较有 retrieval 与无 retrieval 时的预测分布变化，将其视为 signal；再根据该 signal 相对噪声的强弱构造 SNR，决定 guidance scale `g_t`。

# 8. 方法总览

`denoising state → measure retrieval-induced distribution shift → estimate SNR → choose guidance strength → update tokens`。

# 9. 方法细节

ARAM 不是训练一个额外 judge，而是利用模型自身在有/无 context 条件下的分布差异估计 evidence 是否真的提供稳定支持。高 SNR 表示 context 产生集中、可解释的纠正信号；低 SNR 则降低外部 evidence 权重。

# 10. 推理流程

每个关键 timestep 计算 context-conditioned 与 prior-conditioned prediction，得到动态系数，再执行 guidance。

# 11. 实验

在多种 knowledge-intensive QA 上与固定 RAG/guidance baseline 比较，重点验证 robustness 与 overall QA performance。

# 12. 主结果

自适应 guidance 优于固定使用 context 的方案，说明 **evidence trust 应作为 diffusion-time variable**。

# 13. Ablation

值得重点复现：固定 scale vs adaptive scale；clean/noisy/conflicting context；不同 timestep 单独增强/抑制 context。

# 14. Failure Case

如果错误 evidence 与模型 prior 恰好高度一致，distribution shift 可能很小，但两者都错；因此“低冲突”不等于“证据正确”。

# 15. Limitations

- SNR 更多反映 distributional reliability，不直接证明 factual correctness。
- parametric prior 自身可能错误。
- 对 evidence conflict 的细粒度实体级分析不足。

# 16. 与本方向关系

ARAM 对应 **Retrieval for Diffusion Dynamics**：动态控制外部知识对 trajectory 的作用强度。

# 17. Idea Mining

| 项目 | 内容 |
|---|---|
| Assumption | context 应固定强度使用 |
| Phenomenon | retrieval-prior conflict 随 state 变化 |
| Signal | distribution shift SNR、revision、KL |
| Gap | 冲突能否预测最终 factual gain？ |
| Pilot | 按 timestep 注入 clean/noisy/conflict evidence |

# 18. Pilot + Figure

二维热图：横轴 timestep，纵轴 evidence correctness/noise level，颜色为 retrieval gain。若最佳 guidance 区域系统性移动，就可形成 dynamic evidence scheduling 的 motivation。

# 19. 复现价值

2×A100：★★★★★；训练需求：无；idea source：★★★★★。

# 20. 记住什么

1. 外部 evidence 也可能是毒药。
2. evidence utility 是 state-dependent 的。
3. conflict / revision trajectory 很可能是下一代 retrieval controller 的信号。
