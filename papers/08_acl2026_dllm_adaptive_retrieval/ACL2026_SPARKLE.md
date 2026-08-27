# 摘要（中文翻译）

Adaptive RAG 能有效整合外部知识，但已有方法要么依赖冻结 LLM 且缺少显式监督，要么需要昂贵 LLM finetuning。SPARKLE 提出 structured、plug-and-play 的 agentic retrieval policy：额外引入一个 proxy model 独立控制检索过程。proxy 利用 knowledge graph-based reasoning 做结构化 retrieval decision，与 retriever 和主 LLM 解耦，因此可以跨不同 retriever/LLM 泛化。SPARKLE 用 RL 优化，把 retriever 与 LLM 视作环境，并提出 binary-tree rollout 改善探索。三个 in-domain 和四个 out-of-domain QA benchmark 上，相比 adaptive RAG SOTA 平均提升约 9.17% 与 2.85%。

# 1. TL;DR

不一定要改大模型；可以训练一个**小 proxy controller**专门决定 retrieval action。

# 2. 基本信息

- ACL 2026 Long
- Official：https://aclanthology.org/2026.acl-long.1793/
- 标签：`Proxy controller`、`Agentic Retrieval`

# 3. 背景

adaptive RAG 常把 policy 嵌入大 LLM，训练成本高且迁移性差。

# 4. Assumption

> retrieval policy 必须由主生成模型自己学习。

# 5. 核心现象

retrieval control 可以和 generator/retriever 解耦，仍有较好泛化。

# 6. Why

决定“是否/如何搜”所需状态空间远小于完整语言生成。

# 7. Formulation

proxy policy 观察 structured reasoning state，输出 retrieval action；generator/retriever 作为环境。

# 8. 方法

KG reasoning proxy + RL + binary-tree rollout。

# 9. 关键理解

对算力有限团队最有价值：把 innovation 放在 policy/state，而不是重新训练 8B backbone。

# 10. 流程

state → proxy → retrieve/no retrieve/action → environment result → update state。

# 11. 实验

3 in-domain + 4 OOD QA。

# 12. 结果

平均 +9.17% / +2.85%。

# 13. Ablation

structured KG、proxy、tree rollout、RL。

# 14. Failure

proxy state 若缺少关键信号，会做错误 routing；KG 结构不是所有 domain 都有。

# 15. Limitations

仍需 RL；结构化 KG construction 有工程成本。

# 16. dLLM 迁移

最自然实现：冻结 dLLM，抽取 `entropy/revision/stability/entity` 特征，训练极小 retrieval controller。

# 17. Idea Mining

先做 signal pilot；如果 temporal feature 真有预测力，再用小 proxy 学 policy，这样方法从现象自然长出来。

# 18. Pilot

只用 logistic regression / MLP 输入 5–10 个 diffusion signals，预测 retrieve gain；不要一开始上 RL。

# 19. 复现

★★★★☆；对 2×A100 主线高度适配。

# 20. 记住什么

小 controller 是方法终点之一，但前提必须先证明 state signal 真有价值。
