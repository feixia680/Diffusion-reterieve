# 摘要（中文翻译）

RAG 能通过外部知识增强 LLM，但现有研究主要关注 retrieval quality，忽略关键 **integration bottleneck**：即使检索到相关文档，LLM 也可能因与自身参数知识冲突而不能有效利用。本文认为在一次 generation pass 中隐式解决冲突并不理想，因此提出 GuarantRAG，将 reasoning 与 evidence integration 显式解耦。首先只基于参数知识生成 Inner-Answer，捕获模型内部 reasoning；然后生成 Refer-Answer，并用 Contrastive DPO 把 Inner-Answer 当负约束、retrieved documents 当正事实，迫使模型抑制内部 hallucination、忠实抽取外部证据。最后不是简单拼接两者，而用 joint decoding 在 token level 动态融合 Inner-Answer 的逻辑连贯性与 Refer-Answer 的事实精度。五个 QA benchmark 上，GuarantRAG 相比 standard/dynamic RAG 最多提升约 12.1% accuracy，并降低约 16.3% hallucination。

# 1. TL;DR

**检索到正确 evidence ≠ 模型会用。** 真正瓶颈可能在 parametric prior 与 external evidence 的 integration。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.192/
- 标签：`Integration bottleneck`、`Knowledge conflict`

# 3. 背景

多数 RAG pipeline 把 context 拼到 prompt 后就假设模型会 follow evidence。

# 4. Assumption

> relevant document 被放入上下文后，generator 会自然优先相信它。

# 5. 核心现象

参数知识与 retrieved facts 冲突时，LLM 可能忽略外部 evidence 或产生混合 hallucination。

# 6. Why

语言模型在预训练中形成强 prior，一次 forward 中没有明确机制告诉它“这次必须覆盖旧知识”。

# 7. Formulation

显式分开 parametric reasoning 与 evidence-grounded extraction，再在 decoding 阶段融合。

# 8. 方法

Inner-Answer → Contrastive-DPO Refer-Answer → joint decoding。

# 9. 关键理解

retrieval pipeline 应拆成 **retrieve → integrate → generate**，integration 是独立研究对象。

# 10. 流程

parametric answer + retrieved docs → evidence-specific answer → token-level fusion → final。

# 11. 实验

5 个 QA benchmark；accuracy 与 hallucination。

# 12. 主结果

最高 +12.1% accuracy，hallucination -16.3%。

# 13. Ablation

Inner/Refer 分支、Contrastive DPO、joint decoding。

# 14. Failure

训练 evidence branch 成本高；如果 retrieved document 本身错误，强制压制 prior 反而危险。

# 15. Limitations

需要训练；integration policy 与 evidence reliability 尚未完全联合。

# 16. dLLM 迁移

ARAM 研究 distribution conflict，SPREAD 研究 trajectory drift，GuarantRAG 研究 integration bottleneck；三者可统一为 **evidence-prior interaction**。

# 17. Idea Mining

dLLM 的 revision trajectory 能否提前检测 parametric prior 与 evidence conflict？例如 evidence 注入后 entity 来回翻转。

# 18. Pilot + Figure

构造 known conflict QA，注入正确 evidence；画 entity posterior/revision 随 timestep 的变化。比较成功 integration 与失败 integration 的 trajectory pattern。

# 19. 复现

完整：★★★☆☆；conflict pilot：★★★★★。

# 20. 记住什么

1. RAG 的问题不止 retrieve，更重要的是 evidence 是否真正改变 trajectory。
2. 对 dLLM 来说，integration 过程本身是可观察的时间序列。
