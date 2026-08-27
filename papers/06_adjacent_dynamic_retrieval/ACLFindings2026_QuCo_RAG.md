# 摘要（中文翻译）

Dynamic RAG 通过在生成时自适应决定是否检索来降低 hallucination，但已有方法依赖模型内部信号（logits、entropy 等），而 LLM 往往校准很差，甚至会对错误输出表现出很高置信度，因此这些信号并不可靠。QuCo-RAG 从“主观模型置信度”转向“预训练语料的客观统计”。在生成前，它识别低频实体来发现 long-tail knowledge gap；生成过程中，它检查实体在预训练 corpus 中的共现，零共现常提示 hallucination risk。两阶段都使用 Infini-gram 在约 4 万亿 token 上做毫秒级查询，并在 uncertainty 高时触发 retrieval。多跳 QA 上，OLMo-2 相比 SOTA baseline EM 提升约 5–12 点；迁移到训练语料未知的 Llama-3、Qwen2.5、GPT-4.1/5-chat 时最高提升约 14 点，并在长文本和 biomedical QA 上保持效果。

# 1. TL;DR

这篇是我们 future pilot 的重要**负对照**：`internal entropy/confidence ≠ knowledge deficit`。

# 2. 基本信息

- Findings ACL 2026
- Official：https://aclanthology.org/2026.findings-acl.812/
- 标签：`Dynamic RAG`、`Corpus-grounded uncertainty`

# 3. 背景

大量 adaptive RAG 使用 entropy/logit trigger。

# 4. Assumption

> 模型知道自己什么时候不知道。

# 5. 现象

LLM 会 confidently wrong；内部 uncertainty 与 factual knowledge gap 错位。

# 6. Why

神经网络概率是生成分布，不是知识真实性概率；校准受到训练与 decoding 影响。

# 7. Formulation

用 corpus frequency/co-occurrence 构造 objective uncertainty，再决定 retrieve。

# 8. 方法

pre-generation long-tail entity trigger + during-generation co-occurrence verification。

# 9. 方法理解

把“模型自己说不确定”换成“训练世界里这个组合是否有统计支持”。

# 10. 流程

entity detect → corpus query → uncertainty → retrieve/no-retrieve。

# 11. 实验

多跳 QA、long-form、biomedical；多个开放/闭源 LLM。

# 12. 结果

EM +5–12，跨模型最高 +14。

# 13. Ablation

pre-stage vs during-stage、frequency vs co-occurrence、Infini-gram availability。

# 14. Failure

corpus rare 不代表事实错误；新知识/新实体会被误判；闭源模型预训练 corpus 不可得时只能近似迁移。

# 15. Limitations

依赖超大 corpus index；语料统计不是因果知识验证。

# 16. dLLM 迁移

对我们的核心警告：不要假设 entropy 是最佳 retrieval trigger。dLLM temporal signal 必须和真实 intervention gain 比。

# 17. Idea Mining

`corpus rarity × temporal instability` 是否能比任一单变量更准？

# 18. Pilot

构造预测 retrieval gain 的 AUC 表：entropy / revision / entity stability / corpus frequency / combinations。

# 19. 复现

完整：★★★☆☆；小规模 corpus pilot：★★★★★。

# 20. 记住什么

Dynamic Retrieval 的 signal 必须经过真实 retrieval intervention 验证，不能只看“看起来合理”的 uncertainty。
