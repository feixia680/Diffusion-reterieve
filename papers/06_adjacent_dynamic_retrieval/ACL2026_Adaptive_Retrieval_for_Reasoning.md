# 摘要（中文翻译）

本文研究如何利用 adaptive retrieval 为 reasoning-intensive retrieval 找到足够的“bridge documents”。这些文档对推理链有贡献，却未必与初始 query 直接相关。已有 reasoning-based reranker 虽尝试在排序中把 bridge document 提升，但 recall 受限；如果简单地把 adaptive retrieval 加入这类 pipeline，又容易让错误 reasoning plan 在多轮搜索中传播。作者提出 REPAIR，把 reasoning plan 重新用作 adaptive retrieval 的 dense feedback，并允许 reranking 中途进行选择性检索，对关键 plan 做 mid-course correction。复杂检索与 QA 实验中，REPAIR 相比已有方法提升约 5.6 个百分点。

# 1. TL;DR

多跳检索真正难点不是 top-k relevance，而是 **bridge evidence 不直接相关 + 错误 plan 会传播**。

# 2. 基本信息

- 题目：Adaptive Retrieval for Reasoning
- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.1734/
- 方法：REPAIR

# 3. 背景

复杂问题通常要先推理出中间 bridge，再搜下一跳。

# 4. Assumption

> reasoning plan 一旦生成就可以直接用于后续 adaptive retrieval。

# 5. 现象

bridge document 与原 query 低直接相关；错误 plan 被检索器放大，形成 planning error propagation。

# 6. Why

检索结果会反过来影响后续 plan，因此错误形成闭环。

# 7. Formulation

把 plan 当 dense feedback，而非不可修改的 query；允许在 reranking 中选择性追加 retrieval。

# 8. 方法

识别 pivotal plan → retrieval 支持/纠正该 plan → rerank evidence →继续 reasoning。

# 9. 方法理解

不是“每一步都搜”，而是对计划中最需要支持的部分做 mid-course correction。

# 10. 流程

initial retrieve → reasoning plan → rerank → detect gap → selective retrieve → corrected ranking。

# 11. 实验

reasoning-intensive retrieval + complex QA。

# 12. 主结果

相对现有 baseline 约 +5.6pt。

# 13. Ablation

selective retrieval、plan feedback、mid-course correction。

# 14. Failure

若 pivotal plan detection 本身错，仍会搜偏。

# 15. Limitations

依赖 reasoning quality；多轮成本增加。

# 16. dLLM 迁移

dLLM tentative entity 可视为“尚未稳定的 reasoning plan”。SARDI 直接用，而 REPAIR 提醒我们：应该先判断它是否 pivotal/reliable。

# 17. Idea Mining

`tentative bridge entity stability → adaptive retrieval` 是极自然连接。

# 18. Pilot

把 SARDI query 中 entity 按稳定度分组，测低/高稳定实体作为 bridge feedback 的成功率。

# 19. 复现

★★★★☆；对 Retrieval Orchestration baseline 价值★★★★★。

# 20. 记住什么

动态检索最危险的是错误 plan 的自强化；dLLM 中间态也必须防同样问题。
