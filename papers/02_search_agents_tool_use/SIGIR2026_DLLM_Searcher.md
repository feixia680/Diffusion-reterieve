# 摘要（中文翻译）

dLLM 由于并行解码和灵活生成顺序具有独特的效率优势，但 Search Agent 在实践中仍受两类问题限制：其一，ReAct 范式中多轮 reasoning、tool calling 和等待工具返回是串行执行的，端到端延迟很高；其二，现有 dLLM backbone 的推理与工具调用能力较弱，无法直接发挥并行优势。本文提出 DLLM-Searcher。为解决 agent ability 问题，作者设计 Agentic SFT 与 Agentic VRPO 两阶段后训练；为解决 latency 问题，进一步提出 Parallel-Reasoning and Acting（P-ReAct），利用 dLLM 灵活生成顺序优先产出 tool-call 区域，使搜索 API 执行与剩余 reasoning 并行重叠。

# 1. TL;DR

这篇论文利用的不是“dLLM token 更快”，而是 **any-order generation 可以改变 agent 的执行调度**：先把 tool call 生成出来，搜索时模型继续思考。

# 2. 基本信息

- 题目：DLLM-Searcher: Adapting Diffusion Large Language Model for Search Agents
- 作者：Jiahao Zhao 等
- 会议：SIGIR 2026 Full Paper
- Source：https://arxiv.org/abs/2602.07035
- 标签：`Search Agent`、`Tool Use`、`P-ReAct`

# 3. 背景

ReAct 是严格串行：think → call → wait → observe → think。即使模型计算很快，外部工具 latency 仍造成 GPU/CPU idle。

# 4. Existing Assumption

> reasoning token 必须按逻辑文本顺序先生成，tool call 只能等前面的思考完成后再发出。

# 5. 核心现象

- Search Agent 大量延迟来自等待工具，而非模型 FLOPs。
- dLLM 可以先生成后面位置，因此 tool-call region 能提前成熟。
- accuracy-optimal tool timing 与 latency-optimal timing 可能不同。

# 6. 为什么？

AR 解码顺序和输出文本顺序绑定；dLLM 的 denoising 顺序与最终文本顺序解耦，因此可以把“计算依赖”重新调度。

# 7. Problem Formulation

将 agent trajectory 拆成 reasoning region 与 action/tool region，目标同时优化 task reward 与 end-to-end latency。

# 8. 方法总览

- Agentic SFT 学会搜索与推理。
- Agentic VRPO 提升 agent policy。
- P-ReAct 优先完成 tool-call slots，发出搜索；工具运行期间继续 denoise reasoning slots。

# 9. 方法细节

核心系统思想是 overlap：`tool latency` 与 `model reasoning latency` 不再相加，而尽可能取最大值。真正值得研究的变量是 tool call 的 maturity。

# 10. 流程

`query → partial denoise → tool call stable → launch search || continue reasoning → merge observation → final denoise`。

# 11. 实验

重点比较传统 ReAct 与 P-ReAct 的 agent accuracy、search success 与端到端 latency。

# 12. 主结果

论文证明 dLLM flexible-order generation 可以明显降低 agent 等待成本，同时后训练能弥补原始 dLLM tool-use 能力不足。

# 13. Ablation

应重点看：无 P-ReAct、无 SFT、无 VRPO，以及不同 tool-call 提前量。

# 14. Failure Case

过早 tool call 时 query 可能尚未成熟，导致搜错；过晚则失去 latency overlap。

# 15. Limitations

需要 agent post-training；外部 API 延迟分布会影响收益；并非所有任务都能提前确定 tool call。

# 16. 与 Retrieval 的关系

它把 `when to retrieve` 从纯 accuracy 问题扩展为 **accuracy × latency orchestration**。

# 17. Idea Mining

核心 gap：tool-call stability 是否可观测？可以测 query revision、entity stability、action posterior stability。

# 18. Pilot + Figure

横轴“提前多少 denoising steps 发起检索”，纵轴同时画 accuracy 和 latency，寻找 Pareto frontier。

# 19. 复现价值

2×A100：★★★★☆；训练成本：中；agent 实习价值：★★★★★。

# 20. 记住什么

1. dLLM 可以改变 agent schedule。
2. 检索时机不只有正确率，还有外部工具等待成本。
3. query maturity 是最值得继续挖的现象。
