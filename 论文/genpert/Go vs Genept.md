---
title: "Scouter：GO 图（GEARS/bioLORD） vs GenePT（LLM Gene Embedding）"
tags: [scRNA-seq, perturbation, GEARS, biolord, Scouter, GenePT, GO]
aliases: ["GO graph vs GenePT", "Scouter gene embedding 优势"]
---

## 0. 这页笔记解决什么问题？
- **GEARS/bioLORD** 为什么要用 **GO 图**来做未见扰动（unseen perturbation）预测？
- **GenePT（LLM gene embedding）** 和 GO 图在“表示方式/泛化方式/覆盖与工程代价”上有什么本质区别？
- **Scouter 论文**认为 GenePT/LLM embedding 相比 GO 图更优的关键点是什么？

---

## 1. 两种“基因先验”的定义

### 1.1 GEARS/bioLORD：GO 图（gene-gene GO graph）
- Scouter 对 GO 图的描述：**节点是基因**，**边权由共享 GO terms 的数量决定**。[R1]
- GO 图方法的核心表示机制：**每个被扰动基因用其在图中最近邻（nearest neighbors）的组合来表示**（= 借邻居信息做泛化）。[R1]
- GEARS/bioLORD 通过 GO 图构建 **gene perturbation similarity graph**，支持未见扰动外推。[R1]

> 直觉：GO 图把“功能相近”写成显式图结构；模型通过图传播/邻居聚合把未知基因映射到已知空间。

---

### 1.2 GenePT：LLM gene embedding（dense vector）
- GenePT 的基本想法：用 **NCBI 基因文本描述**（functional summary 等）+ LLM embedding 得到每个基因的向量表示。[R2]
- Scouter 使用的 GenePT embedding：OpenAI `text-embedding-ada-002` 生成，**1536 维**。[R1]
- **重要实现点**：Scouter 把这些 gene embeddings 作为输入的一部分，**训练时保持不变（冻结，不更新）**。[R1]

> 直觉：GenePT 把“基因语义/功能关系”压到连续向量空间，基因相似性可直接用向量距离/相似度度量。

---

## 2. 本质区别：图结构 vs 向量空间

### 2.1 “距离”从哪里来？
- **GO 图**：相似性主要通过“是否相连、边权多大、邻居是谁”体现；“距离”往往要靠图结构（最短路/邻居）间接表达。Scouter 强调其依赖 nearest neighbors 的组合表示。[R1]
- **GenePT**：天然是连续空间，距离/相似度可直接算（cosine/欧氏等）；Scouter 直接把它作为 embedding 输入。[R1]

### 2.2 unseen perturbation 泛化机制不同
- **GO 图泛化**：未知基因如果在图里有足够“可靠邻居”，可以借邻居传播；但如果图很稀疏或该基因缺失，泛化会崩。[R1]
- **GenePT 泛化**：只要该基因有文本描述 -> 就能生成 embedding -> 在同一向量空间中定位 -> 可支持未见基因输入。[R2]

---

## 3. Scouter 论文强调：GenePT 相比 GO 图的优势（重点）

### 3.1 GO 图 **稀疏（sparsity）**：很多基因对根本没有有效连接
- Scouter 明确指出：GO 图“高度稀疏”，只有很小比例的基因对共享 GO terms，影响预测的准确性和可靠性。[R1]

**理解要点**
- 你想靠“邻居借信息”，但邻居少/边弱 -> 信息传播有限 -> unseen 基因更难靠图泛化。

---

### 3.2 GO 图 **覆盖不全（incomplete gene coverage）**：有些基因直接“不可用”
- Scouter 指出：一些基因被排除在 GO 图之外，导致涉及这些基因的扰动结果无法预测。[R1]
- 论文给出案例：当基因不在 GO 图中时，GO 图方法无法预测，而 Scouter 仍可预测。[R1]

**理解要点**
- GO 图路线的失败是“结构性不可用”（没有节点/没有边），而不是“效果差一点”。

---

### 3.3 GenePT/LLM embedding 是 **稠密、信息更丰富** 的先验
- Scouter 在方法段说明：他们用 LLM gene embeddings（GenePT）替代 GO 图作为基因表示来源，并将其固定作为输入。[R1]
- GenePT 论文强调：文本 embedding 是一种简单有效、无需额外预训练的数据高效路径。[R2]

**理解要点**
- embedding 的信息来源是“文本知识”（NCBI summary/文献总结等），不依赖 GO 图的显式注释连边，因此理论上更不怕 GO 覆盖缺失。

---

### 3.4 工程可用性：Scouter 更“易用、低门槛”
- Scouter 摘要/方法强调：无需预训练、在标准硬件上高效运行，并且 embedding 预先生成即可。[R1]
- GenePT 提供可下载的 embeddings（包含 NCBI summary 与对应 OpenAI embeddings），降低复现门槛。[R3]

---

## 4. 重要补充：Scouter 也强调“不是仅仅换 embedding 就行”
- Scouter 做了对照：把 GEARS/bioLORD 的 GO embedding 替换成 LLM embedding 只能带来有限提升，说明 Scouter 的收益还来自其训练/结构设计（如 compressor–generator、采样策略等）。[R1]

---

## 5. 对你复现/改模型的直接启示（实践）
1. **如果你的任务遇到 GO 覆盖缺失**（目标基因不在 GO 图）：GO 图路线可能直接无法预测；GenePT/LLM embedding 更稳健。[R1]
2. **如果你只“把 GO embedding 换成 LLM embedding”**，未必能复制 Scouter 的增益；还需要匹配它的训练与网络设计（Scouter 已做对照说明）。[R1]
3. **实现层面**：GenePT embedding 是“查表特征”，训练时冻结；工程更像标准 MLP/条件生成网络，而不是 GNN/图采样管线。[R1]

---

## 6. 一句话结论
- **GO 图（GEARS/bioLORD）**：结构化、可解释，但受 **稀疏性 + 覆盖不全**限制，甚至出现“某些基因不可用”。[R1]
- **GenePT（Scouter）**：把文本知识压成稠密向量，天然可度量相似性，减少对 GO 图结构与覆盖的依赖；Scouter 用它作为关键先验并展示更好的 unseen 泛化与可用性。[R1]

---

## 参考链接
- [R1] Scouter (Nature Computational Science, 2025): `https://www.nature.com/articles/s43588-025-00912-8`
- [R2] GenePT (bioRxiv, 2023/2024 preprint): `https://www.biorxiv.org/content/10.1101/2023.10.16.562533v2`
- [R3] GenePT embeddings (Zenodo): `https://doi.org/10.5281/zenodo.10833191`
