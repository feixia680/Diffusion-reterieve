# 摘要（中文翻译）

DLM 提供并行序列生成与灵活 token 顺序，但推理仍慢于 AR 模型，主要因为双向注意力和大量 refinement steps。本文发现一个被忽略的性质：**early answer convergence**。在很多样本中，正确答案在最终 decoding 前很早已经可内部识别，在 semi-autoregressive 与 random remasking schedule 下都成立。例如 GSM8K 和 MMLU 上分别最高约 97% 和 99% 的样本只用一半 refinement steps 就可正确解码。基于此，作者提出 training-free 的 Prophet：根据 top-2 prediction confidence gap 动态决定继续 refinement，还是直接 all-in 解码所有剩余 token。LLaDA-8B、Dream-7B 多任务实验表明，Prophet 最多减少约 3.4× decoding steps，同时保持生成质量。

# 1. TL;DR

最值得学的是 paper story：**先发现“答案已经知道，只是还没完全解码”→ 再做 early stop**。

# 2. 基本信息

- 题目：Diffusion Language Models Know the Answer Before Decoding
- ICLR 2026 Oral
- Source：https://arxiv.org/abs/2508.19982
- 标签：`Early convergence`、`Stopping`

# 3. 背景

DLM 速度慢往往被归因于必须完成固定 refinement steps。

# 4. Existing Assumption

> 最终答案只有到接近最后 timestep 才可靠形成。

# 5. 核心现象

大量样本在 half steps 已经能得到最终正确答案；后半程很多计算只是“形式上的收敛”。

# 6. Why

关键语义/答案 token 的 posterior 可能较早拉开 top-1/top-2 gap，剩余迭代主要修饰低价值位置。

# 7. Problem Formulation

把 decoding 转成 optimal stopping：何时继续 refinement 的期望收益小于计算成本？

# 8. 方法

Prophet 监控 top-2 confidence gap；达到条件时 all-in decode remaining masks。

# 9. 公式理解

confidence gap 是 `p_1-p_2` 的稳定性 proxy；gap 大表示当前最优预测相对第二候选有明显优势。

# 10. 流程

每步预测 → 计算 gap → continue 或 all-in。

# 11. 实验

LLaDA-8B、Dream-7B；GSM8K、MMLU 等；比较 steps reduction 与 quality。

# 12. 主结果

最高 3.4× 减步，half-step correct decode 比例极高。

# 13. Ablation

重点是不同阈值、不同 remasking schedule、不同任务的 early-convergence rate。

# 14. Failure Case

confidence gap 大也可能是 confidently wrong；QuCo-RAG/UNCODE 给出了很好的反例视角。

# 15. Limitations

early answer convergence 不等于 early knowledge sufficiency；不能直接推出何时无需 retrieval。

# 16. Retrieval 关系

最自然的问题：**retrieval need 是否也 early converge？** 若当前 trajectory 已稳定正确，继续检索可能只增加噪声。

# 17. Idea Mining

将 answer stability、entity stability、retrieval gain 放在同一时间轴上，研究三者谁先收敛。

# 18. Pilot

逐 timestep 计算无检索答案、检索后答案与最终答案，画 `t → retrieval gain`，看最佳 retrieval window 是否先升后降。

# 19. 复现价值

2×A100：★★★★★；training-free；idea source：★★★★★。

# 20. 记住什么

1. dLLM 的“语义完成”可能远早于“解码完成”。
2. retrieval 也应研究最佳停止时机。
