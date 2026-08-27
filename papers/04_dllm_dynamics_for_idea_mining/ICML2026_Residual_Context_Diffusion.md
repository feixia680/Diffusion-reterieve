# 摘要（中文翻译）

dLLM 能并行解码多个 token，但当前 block-wise dLLM 通常依赖 remasking：只保留最有置信度的 token，丢弃其他预测，造成计算浪费。作者发现被丢弃 token 仍保留对后续解码有用的上下文信息，因此提出 Residual Context Diffusion（RCD），把 discarded token representations 转换为 contextual residual，并在下一 denoising step 注入。为避免反向传播的显存瓶颈，采用解耦的两阶段训练。RCD 在长 CoT 的 SDAR 与短 CoT 的 LLaDA 上验证，只需约 3 亿 token 即可把标准 dLLM 转成 RCD；多个 benchmark 上准确率提升 4–11 个百分点，额外计算很小，AIME 上接近翻倍，并在 baseline 峰值准确率下最多减少约 4–5× denoising steps。

# 1. TL;DR

RCD 与 SARDI 形成关键互证：**discarded prediction 真的含有未来有用信息**。RCD 内部复用，SARDI 外部检索。

# 2. 基本信息

- ICML 2026
- Source：https://arxiv.org/abs/2601.22954
- Code：https://github.com/yuezhouhu/residual-context-diffusion

# 3. 背景

remask 机制把大量已计算 distribution 直接丢掉。

# 4. Assumption

> 未被 commit 的预测没有后续价值。

# 5. 现象

discarded token representations 保留有助于后续 reasoning 的 context。

# 6. Why

“不能提交”只代表不够确定，不代表没有 partial semantic information。

# 7. Formulation

将 discarded state `h_t^discard` 映射为 residual `r_t`，注入下一步 hidden/context。

# 8. 方法

Residual module + two-stage training，复用原本浪费的前向信息。

# 9. 方法理解

它类似 temporal memory：上一 step 的“失败候选”不直接成为 token，而作为 soft context 留给下一步。

# 10. 流程

predict → commit confident tokens → encode discarded representations → residual → next denoise。

# 11. 实验

SDAR、LLaDA，多 reasoning benchmarks。

# 12. 结果

4–11pt accuracy gain；AIME 显著提升；可减少 denoising steps。

# 13. Ablation

residual source、训练 token 数、注入方式、不同 step。

# 14. Failure

错误 residual 也可能累计；过强 temporal memory 可能抑制必要 revision。

# 15. Limitations

需要额外训练；没有直接区分 useful vs harmful discarded info。

# 16. Retrieval 关系

SARDI 是 `discarded state → external retrieval`，RCD 是 `discarded state → internal memory`。可以研究二者何时互补。

# 17. Idea Mining

比较同一 discarded state 的 internal reuse gain 与 external retrieval gain，判断模型到底缺“推理上下文”还是“外部知识”。

# 18. Pilot

2×2：RCD on/off × Retrieval on/off，按问题知识需求分组，观察交互效应。

# 19. 复现

★★★★☆；轻量训练；idea source ★★★★★。

# 20. 记住什么

被丢弃状态是一种可复用资源，关键不是“用不用”，而是“内部复用还是外部检索”。
