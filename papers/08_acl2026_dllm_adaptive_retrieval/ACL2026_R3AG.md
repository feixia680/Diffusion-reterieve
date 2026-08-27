# 摘要（中文翻译）

RAG 已成为知识密集任务的重要方法，但 one-size-fits-all retrieval 是明显瓶颈：不同 query 对不同 retriever 有不同偏好。已有 routing 方法尝试动态选择 retriever，却通常建立在“单一且静态 capability”假设上，仅依据 semantic relevance 做选择。本文指出 RAG 中存在关键区分：retrieved document 不仅要 relevant，还必须真正帮助 generator 生成正确答案。R³AG 显式建模 query 与 retriever capability 的动态对齐，把 capability 分解成两维：retrieval quality 与 generation utility。通过 contrastive learning，同时使用 document assessment 与 downstream answer correctness 两类互补监督，学习 query-specific preference shift。多种 knowledge-intensive task 上，R³AG 持续超过最佳单 retriever 与 SOTA routing 方法。

# 1. TL;DR

**Relevance ≠ Generation Utility。** 检索器选得“看起来相关”不等于最终回答更好。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.939/
- 标签：`Retriever Routing`、`Generator Utility`

# 3. 背景

routing 常用 Recall/relevance 作为唯一监督。

# 4. Assumption

> retriever relevance 高，generator 就自然受益。

# 5. 核心现象

同一 relevant evidence 在不同 generator/query 下 downstream utility 不同。

# 6. Why

generator 有自己的参数知识、attention bias、integration ability；它可能不会用、甚至被 evidence 干扰。

# 7. Formulation

retriever capability = retrieval quality + generation utility；学习 query-conditioned router。

# 8. 方法

contrastive learning with document relevance + final answer correctness signals。

# 9. 关键理解

retrieval evaluation 必须走到最终 generator intervention。

# 10. 流程

query → router → select retriever → documents → generator → answer。

# 11. 实验

多 knowledge-intensive QA/retrieval tasks。

# 12. 主结果

优于 best individual retriever 与 static/SOTA routing。

# 13. Ablation

只用 relevance、只用 generation utility、联合监督。

# 14. Failure

generation utility label 依赖具体 generator，换模型后可能 shift。

# 15. Limitations

router 对 generator/corpus drift 的迁移值得进一步研究。

# 16. dLLM 迁移

我们定义 retrieval gain 时必须用 **final dLLM answer improvement**，不能只看 Recall@k。

# 17. Idea Mining

某 tentative query 的 retrieval recall 可能高，但 evidence 会造成 priming/drift；应显式建模 trajectory utility。

# 18. Pilot

每个 retrieval event 同时记录 Recall/relevance 与 final gain，计算两者 mismatch rate；找出“高 relevance 负 gain”样本的 diffusion dynamics。

# 19. 复现

★★★★★；Routing baseline 强烈推荐。

# 20. 记住什么

以后所有 retrieval signal 的 ground truth 都应尽量定义成 downstream gain，而不是检索器自己的分数。
