# 摘要（中文翻译）

Masked DLM 依赖 masking/unmasking，计算效率与生成灵活性受该范式限制。本文提出 Deletion-Insertion Diffusion（DID），严格把 token deletion 与 insertion 表述为离散 diffusion forward/backward process，用它们替代 mask/unmask。DID 消除两类主要无效计算：大量非信息 `<MASK>` token，以及 variable-length 场景中的 `<PAD>`；同时原生支持可变长度序列，并因 insertion 动态调整 token position 而具备内在 positional self-correction。训练上作者设计 insertion score 和对应 score-based objective，涉及 subsequence counting，并用并行 dynamic programming 高效求解。固定/可变长度实验显示，DID 在 modeling、sampling quality、训练/推理速度上优于 MDLM 与 insertion-based baselines。

# 1. TL;DR

DID 把 dLLM 的“自我修改”从 token substitution 扩展到 **删除/插入/位置修正**。

# 2. 基本信息

- ICLR 2026
- 标签：`Deletion-Insertion Diffusion`、`Self-correction`

# 3. 背景

mask canvas 固定长度且大量无效 `<MASK>/<PAD>`。

# 4. Assumption

> diffusion text generation 必须在固定 slot canvas 上 mask/unmask。

# 5. 现象

insertion/deletion 可以作为合法 diffusion transition，并天然支持 variable length/self-correction。

# 6. Why

文本结构不是固定位置对象；允许位置重排更符合编辑式生成。

# 7. Formulation

forward = deletion；reverse = insertion；学习 insertion score。

# 8. 方法

DID + score-based training + parallel DP subsequence counting。

# 9. 关键理解

“state”不仅包括 token identity，还包括 token existence/position。

# 10. 流程

delete-corrupt training sequences → learn insertion score → reverse insertion generation。

# 11. 实验

fixed/variable length、generation quality、speed。

# 12. 结果

多个维度优于 masked diffusion baseline。

# 13. Ablation

insertion score、DP、variable-length handling。

# 14. Failure

结构更复杂，难直接兼容现有 masked-dLLM RAG pipeline。

# 15. Limitations

需要重新训练模型；不是 training-free。

# 16. Retrieval 关系

未来 retrieval 不一定只“改 token”，也可以触发局部 insertion：把新 evidence 作为显式新增 reasoning span。

# 17. Idea Mining

retrieved fact 应替换旧 span，还是插入新 span？不同 evidence type 的最佳 edit operator 可能不同。

# 18. Pilot

在 masked dLLM 中近似模拟 replace vs insert evidence prompts，比较冲突修复能力。

# 19. 复现

★★☆☆☆；完整训练成本高。

# 20. 记住什么

dLLM 的独特性是“可修订的全局状态”，不必局限于 mask/unmask。
