# 摘要（中文翻译）

DLM 迭代去噪时不断决定哪些位置可以提交。标准方法贪心 unmask 最有置信度的位置，但局部选择可能锁入次优顺序，尤其在复杂推理 prompt 上。SOAR 是 training-free decoding 方法，根据模型 uncertainty 自适应切换行为：低置信时临时扩大对不同 unmasking 决策的搜索，避免过早 commitment；高置信时收缩搜索并并行解码更多位置，减少去噪迭代。Dream-7B、LLaDA-8B 在 GSM8K、MBPP、HumanEval 上表明，SOAR 在保持有竞争力速度的同时提升生成质量。

# 1. TL;DR

SOAR 的核心是 **state-dependent action switching**：不确定就 search，确定就 accelerate。

# 2. 基本信息

- ICML 2026
- Source：https://arxiv.org/abs/2602.10953
- 标签：`Adaptive compute`、`Search/Accelerate`

# 3. 背景

固定 decoding policy 对所有状态一视同仁。

# 4. Assumption

> 同一个 confidence-based unmask rule 在全 trajectory 都合适。

# 5. 现象

低置信状态需要探索，高置信状态可以激进并行；两者使用同策略会同时损失质量和速度。

# 6. Why

不同状态的 decision risk 不同，最优 test-time compute 应自适应。

# 7. Formulation

`state s_t → {search alternatives, accelerate parallel decoding}`。

# 8. 方法

confidence-switched position beam search。

# 9. 关键理解

它已经接近一个小型 controller：观测 uncertainty，分配 compute budget。

# 10. 流程

低 confidence → beam over position choices；高 confidence → collapse beam / commit more positions。

# 11. 实验

Dream-7B、LLaDA-8B；数学+代码。

# 12. 结果

quality-speed tradeoff 优于固定 greedy decoding。

# 13. Ablation

切换阈值、beam size、并行位置数。

# 14. Failure

confidence calibration 不可靠时 controller 会错切换。

# 15. Limitations

仍把 confidence 当主要 state；未引入 temporal stability 或 external knowledge need。

# 16. Retrieval 关系

把 action space 改为 `{retrieve, internal search, parallel decode, stop}`，就是 dLLM Retrieval Orchestration 的雏形。

# 17. Idea Mining

研究何时应该花 test-time budget 做 internal search，何时应该付 external retrieval cost。

# 18. Pilot

按 confidence × retrieval gain 分四象限，专门找“低 confidence 但 retrieval 无效”和“高 confidence 但 retrieval 有效”的反例。

# 19. 复现

★★★★★，训练免。

# 20. 记住什么

固定策略不是最优；dLLM 可以把内部搜索与外部检索统一成 state-dependent compute allocation。
