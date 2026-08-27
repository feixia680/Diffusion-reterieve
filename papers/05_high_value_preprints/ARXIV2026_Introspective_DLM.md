# 摘要（中文翻译）

DLM 虽能并行生成，但生成质量仍落后 AR 模型。本文将这一差距归因于 **introspective consistency failure**：AR 模型通常认可自己先前生成的 token，而 DLM 经常在后续步骤中“不认可”自己的历史预测。作者定义 introspective acceptance rate，用于衡量模型是否接受自己之前产生的 token，并据此解释 AR 训练的结构优势：causal mask 和 logit shift 会隐式强化这种自洽性。基于该观察，作者提出 Introspective Diffusion Language Model（I-DLM），保留 diffusion-style parallel decoding，同时继承 AR training 的 introspective consistency；其 ISD（introspective strided decoding）可在同一 forward 中验证旧 token 并推进新 token。系统上进一步利用 AR 继承优化与 stationary-batch scheduler。I-DLM 在 15 个 benchmark 上同时提升质量和 serving efficiency，并在 AIME-24、LiveCodeBench 等取得强结果。

# 1. TL;DR

**revision / self-rejection 本身是可观测信号**：模型是否愿意接受自己刚刚生成的 token，能够揭示 trajectory 稳定性。

# 2. 基本信息

- 状态：arXiv 2026
- Source：https://arxiv.org/abs/2604.11035
- 标签：`Introspection`、`Self-rejection`、`Revision`

# 3. 背景

传统 DLM analysis 常看 confidence，却很少问“模型下一个 step 是否还认同上一步”。

# 4. Assumption

> 每一步重新预测只是正常 refinement，不需要把跨步自洽性单独建模。

# 5. 现象

DLM 对自己早先 token 的 acceptance 显著弱于 AR，且这和质量差距相关。

# 6. Why

AR training 的 causal shift 天然迫使后续 token 在已生成 prefix 条件下自洽；masked diffusion 没有同等约束。

# 7. Formulation

定义 introspective acceptance：固定先前生成 token，重新条件预测时是否仍支持它。

# 8. 方法

I-DLM + ISD，把验证过去与生成未来合并到一次 forward。

# 9. 方法理解

核心不是“不允许 revision”，而是让 revision 更有意义：真正错误时改，正确时保持。

# 10. 流程

parallel generate → introspective verify → retain/revise → stride forward。

# 11. 实验

15 benchmarks，含数学、代码、服务效率。

# 12. 结果

质量接近同尺度 AR，同时 serving 更高效；AIME-24 69.6、LiveCodeBench-v6 45.7。

# 13. Ablation

acceptance rate 与质量、ISD、scheduler 等。

# 14. Failure

高自洽也可能是稳定 hallucination；self-acceptance 不能等同 factual correctness。

# 15. Limitations

需要新的训练/模型范式；对现成 dLLM training-free 应用不直接。

# 16. Retrieval 关系

**self-rejection / revision frequency** 是比单步 entropy 更自然的 temporal knowledge-conflict signal。

# 17. Idea Mining

测 span-level introspective rejection 是否预测 external retrieval gain，尤其 entity/relation span。

# 18. Pilot

对每个 tentative entity 统计后续 N 步的 acceptance ratio，与“检索后是否纠正最终答案”做相关分析。

# 19. 复现

完整模型：★★☆☆☆；只做 signal pilot：★★★★★。

# 20. 记住什么

1. temporal self-consistency 是 dLLM 独特变量。
2. self-rejection 很值得作为 retrieval trigger 候选。
