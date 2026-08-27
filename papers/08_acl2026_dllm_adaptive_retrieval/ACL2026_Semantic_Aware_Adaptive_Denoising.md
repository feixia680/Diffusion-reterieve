# 摘要（中文翻译）

DLM 是 GPT-style 顺序生成之外的强非自回归方案，但迭代并行 denoising 带来巨大计算开销。已有加速方法难以准确识别语义已经稳定的 token，因此实际 speedup 有限。本文首次系统研究 DLM 的 convergence dynamics，发现传统 scalar detection criterion 与真正的 semantic convergence 存在错位，并观察到 post-peak confidence：某些 token 在达到最有效置信状态后继续计算，不仅浪费 denoising compute，甚至降低质量。作者提出 Ada-DLM，把 scalar confidence trajectory 编码为 evolution-aware feature vector，并通过 clustering 主动识别 semantically converged tokens，同时加入系统优化。实验上相对 SOTA 最多约 2× speedup，并有最高约 19% quality improvement。

# 1. TL;DR

**单点 confidence ≠ semantic convergence；要看 confidence trajectory 的形状。**

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.819/
- 标签：`Semantic convergence`、`Temporal feature`

# 3. 背景

现有 stopping/caching 常用当前 confidence 阈值。

# 4. Assumption

> 一个 scalar confidence 足以判断 token 是否收敛。

# 5. 核心现象

- scalar criterion 与 semantic convergence misaligned。
- post-peak confidence 阶段可能继续算反而降质。

# 6. Why

收敛是时间过程，单个 snapshot 丢失“此前如何变化”的信息。

# 7. Formulation

把多步 confidence 序列编码成 evolution-aware vector，再识别 converged cluster。

# 8. 方法

Ada-DLM：temporal feature encoding + clustering + runtime optimization。

# 9. 关键理解

需要看 derivative/history，而不只是 absolute confidence。

# 10. 流程

collect confidence history → vectorize → cluster → stop/skip converged tokens。

# 11. 实验

多个 DLM inference setting；quality + runtime。

# 12. 结果

最高约 2× speedup、19% quality improvement。

# 13. Ablation

temporal feature、cluster strategy、system optimization。

# 14. Failure

语义收敛也可能收敛到错误实体。

# 15. Limitations

聚类规则可能 model/task dependent；不判断 factual correctness。

# 16. Retrieval 关系

这是比 SureLock 更进一步的 signal：**trajectory shape** 可能比 current posterior 更能判断 retrieve/stop。

# 17. Idea Mining

用 confidence evolution vector 预测 retrieval gain，而不是用单步 entropy。

# 18. Pilot

输入最近 k 步的 `[entropy, top2-gap, revision]` 到一个极小 classifier，预测 retrieval intervention 是否正收益；与单步阈值比较。

# 19. 复现

★★★★★；可先做 training-free feature study。

# 20. 记住什么

retrieval trigger 很可能需要 temporal feature，而不是 scalar threshold。
