# 摘要（中文翻译）

本文为 DLM 的并行生成优势提供严格理论基础。作者形式化 parallel sampling 模型，并证明：带多项式长度 CoT 的 DLM 可以用最优数量的 sequential steps 模拟任意 parallel sampling algorithm。因此，只要目标分布可以用少量顺序步骤生成，DLM 也能以同样的最优 sequential complexity 生成。进一步地，如果不允许修改已经 reveal 的 token，即使有 CoT 也可能产生很大的中间空间开销；而允许 remasking（把已 reveal token 重新变 mask）或 revision（改成其他 token）后，DLM 可用最优空间复杂度模拟任意 parallel sampler。论文还证明 revision/remasking 带来严格 expressivity gap：允许修改的 DLM 严格强于不能修改的 DLM。

# 1. TL;DR

revision 不是实现细节，而是 **DLM 理论能力的一部分**。

# 2. 基本信息

- ICLR 2026
- 作者：Haozhe Jiang, Nika Haghtalab, Lijie Chen
- Official：ICLR proceedings
- 标签：`Theory`、`Revision`

# 3. 背景

很多加速方法试图减少 revision，认为它只是浪费。

# 4. Assumption

> 越少 remask/revision 越好，最好一次 commit。

# 5. 核心现象/定理

允许 revision/remasking 带来严格表达能力与空间复杂度优势。

# 6. Why

并行算法需要更新中间共享状态；固定已 reveal token 会把早期决定永久冻结，迫使模型用更多额外空间补救。

# 7. Formulation

用 sequential steps 与 intermediate footprint 衡量 parallel sampler complexity。

# 8. 方法

理论 simulation 与 expressivity proof。

# 9. 核心理解

revision 本质是“撤销错误/暂定决策”的计算原语。

# 10. 推理意义

设计 dLLM 系统时，不应把 revision rate 一概视为低效。

# 11. 实验

理论论文为主。

# 12. 结论

支持 revision 的 DLM 可以是 optimal parallel sampler。

# 13. Controlled Question

经验上应验证：哪些 revision 是有益自纠，哪些是知识冲突/不稳定。

# 14. Failure

理论最优不意味着真实网络训练后能学到理想 revision policy。

# 15. Limitations

抽象模型与大规模 DLM 的 calibration 仍有距离。

# 16. Retrieval 关系

revision frequency 之所以有研究价值，不是因为它一定坏，而是它携带“模型正在重新决定”的状态信息。

# 17. Idea Mining

区分 beneficial revision 与 pathological revision；只有后者可能需要 external retrieval。

# 18. Pilot

把 revisions 按最终是否纠正答案分成 helpful/harmful，测试两类的 entropy、entity stability 与 retrieval gain。

# 19. 复现

理论阅读★★★★★；实验复现不必要。

# 20. 记住什么

不要直接提出“减少 revision”；先理解 revision 的语义类型。
