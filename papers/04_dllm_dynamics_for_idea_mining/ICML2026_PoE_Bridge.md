# 摘要（中文翻译）

DLM 通过并行解码具有速度优势，但缺少 token dependency，使生成质量落后 AR 模型。已有方法用 importance sampling 让 DLM 作为 proposal、AR 作为 target，但二者分布差距很大，需要大量 particles，计算昂贵。PoE-Bridge 引入中间分布来桥接二者，该分布是 DLM proposal 与 AR target 的 Product-of-Experts。首先用 DLM 并行 draft 多个 continuation，再用 rejection sampling 验证并使候选靠近 PoE；随后用 importance sampling 进一步修正到 AR target。还加入 mixed-temperature sampling 与 elastic rejection windows。实验上 PoE-Bridge 相比标准 DLM decoding 取得约 5× speedup，同时恢复至少 95% 的目标 AR 性能，在数学与代码任务上明显缩小质量差距。

# 1. TL;DR

当两个分布差得太远，**不要一步纠正，先构造 bridge distribution**。

# 2. 基本信息

- ICML 2026
- Source：https://arxiv.org/abs/2606.08048
- 标签：`Verification`、`Product of Experts`

# 3. 背景

DLM 快但 quality 低；AR 慢但 distribution 更可靠。

# 4. Assumption

> DLM proposal 可以直接通过 importance sampling 对齐 AR target。

# 5. 现象

proposal-target gap 太大导致粒子效率极低。

# 6. Why

高维序列分布重叠区域小，importance weights 高方差。

# 7. Formulation

构造中间 PoE distribution，逐步把 DLM sample 移向 AR target。

# 8. 方法

DLM parallel draft → rejection toward PoE → importance correction toward AR。

# 9. 关键理解

bridge 的价值是降低一次性 distribution correction 难度。

# 10. 流程

proposal、verify、bridge、correct。

# 11. 实验

数学 reasoning 与 coding。

# 12. 主结果

约 5× speedup，恢复至少 95% AR target performance。

# 13. Ablation

PoE、temperature mixing、elastic window、particle count。

# 14. Failure

需要额外 AR verifier，部署复杂度与显存增加。

# 15. Limitations

并非纯 dLLM；更偏 quality-speed bridging。

# 16. Retrieval 关系

external evidence 也可视为另一个 expert。未来可研究 `DLM prior × retrieval evidence` 的 bridge，而不是硬拼 context。

# 17. Idea Mining

ARAM 处理 retrieval-prior conflict，PoE-Bridge 处理 model-distribution gap；二者可统一成“多 expert gradual integration”。

# 18. Pilot

构造 correct/conflicting evidence，测一次性强 guidance 与渐进 evidence guidance 的 trajectory stability。

# 19. 复现

★★★☆☆；需要 AR+DLMM；idea source ★★★★☆。

# 20. 记住什么

如果 retrieval evidence 与 prior 冲突很大，逐步 bridge 可能比硬注入更自然。
