
---
title: "GenePert：用 GenePT embedding + 岭回归预测基因扰动效应（Method 重点 Obsidian 笔记）"
tags: [paper, perturb-seq, crispr, single-cell, perturbation, genpert, genept, ridge, l2, bias-variance, esm2, deshift, ot]
date: 2026-01-25
---

## 1. 这篇论文在解决什么问题（一句话）
给定单细胞扰动数据（Perturb-seq），希望学习一个模型：**输入 = 扰动基因（或基因对）**，输出 = **扰动后的平均表达响应（pseudo-bulk）或相对对照的平均扰动效应**，并能对**训练中未出现过的扰动基因**做预测。

---

## 2. 你问过的基础概念

### 2.1 Perturb-seq 是什么
Perturb-seq = **大规模基因扰动**（通常通过 CRISPR） + **单细胞 readout**（常见是 scRNA-seq）。
- 每个细胞既有表达谱（RNA count/表达向量），又能知道它被哪个 gRNA/基因扰动。
- 目标是用这些数据学习扰动如何改变基因表达。

### 2.2 CRISPR / CRISPRi / CRISPRa 是什么
- CRISPR（常指 CRISPR-Cas9）：直接切 DNA，常用于敲除（knock-out）。
- CRISPRi：interference，使用 dCas9（不切 DNA）+ 抑制结构域，使目标基因表达下降（抑制）。
- CRISPRa：activation，使目标基因表达上升（激活）。

---

## 3. 数据与 `adata.X`：怎么测量出来、代表什么（你问过）

### 3.1 `adata.X` 的来源（实验层面）
`adata.X` 来源于单细胞 RNA 测序（scRNA-seq）：
1. 每个细胞的 mRNA 被捕获并测序；
2. 读段（reads）比对到基因，得到每个细胞-每个基因的计数（counts）；
3. 这些 counts 经常要做归一化和变换，才作为机器学习输入。

### 3.2 论文里构造输出矩阵 X 的预处理（关键）
论文在 dataset 处理部分给出典型流程：
- 每个细胞按总 counts 归一化到 10,000；
- 然后做 log1p：$x\mapsto \log(1+x)$；
- 为降低维度与稀疏性，仅保留最“高变”的 5,000 个基因（HVGs），即 $q=5000$。

> 直观：`adata.X[i, g]` 是“经过 library size normalization + log1p 的表达值”，用于可比建模。

---

## 4. 预测目标：平均扰动效应（Method 前置）

很多评估不直接看绝对表达，而看相对对照的效应（relative scale）：
- 对每个扰动 $p$，先把该扰动下细胞表达取平均（pseudo-bulk）得到 $\bar{X}_p$；
- 对对照细胞取平均得到 $\bar{X}_c$；
- 预测目标常写为：

$$
\Delta_p = \bar{X}_p - \bar{X}_c
$$

其中 $\Delta_p \in \mathbb{R}^q$，是“扰动导致的平均表达变化模式”。

---

## 5. Method：GenePert 的建模（重点）

### 5.1 核心思想
GenePert 的核心不是复杂网络，而是：
1) 用一个可泛化的向量表示“扰动是什么”（GenePT embedding）  
2) 用一个简单、稳定的预测器（岭回归）学习从 embedding 到平均效应的映射

---

## 6. Method 细节 1：GenePT embedding（你问过：与普通 embedding 区别、表征是什么）

### 6.1 GenePT 是什么
GenePT 给每个基因 $p$ 一个向量表示：

$$
g_{\text{GenePT}}(p)\in\mathbb{R}^d
$$

论文主实验里 $d=3072$（来自文本 embedding 模型）。

GenePT 的信息来源是**基因的功能/事实描述文本**（如 NCBI、UniProt 的 curated summaries），经 LLM embedding 得到。

### 6.2 GenePT vs “普通 embedding”的区别（最关键的三点）
1) **知识来源不同**：GenePT 来自知识库文本语义；很多“普通 embedding”来自表达数据/序列/图或随机可学习向量。  
2) **对象层级不同**：GenePT 是 gene-level（扰动条件级）；VAE latent 是 cell-level（细胞级）。  
3) **未见扰动友好**：只要有文本，就能算新基因的 embedding，不要求该基因在训练扰动中出现过。

### 6.3 表征（representation）是什么（你问过）
表征就是把原始对象（基因/细胞/药物）变成向量，以便模型学习。
- GenePert 的表征：基因语义向量 $g_{\text{GenePT}}(p)$
- DeShift/VAE 的表征：细胞 latent $z_i$

---

## 7. Method 细节 2：输入归一化（单位 L2）
论文对输入特征做 L2 归一化，使不同基因 embedding 在同一尺度：

$$
\tilde g(p)=\frac{g_{\text{GenePT}}(p)}{\|g_{\text{GenePT}}(p)\|_2}
$$

意义：
- 避免某些 embedding 因“模长更大”而在回归中被不公平放大；
- 与后续的 L2 正则（岭回归）搭配更稳定。

---

## 8. Method 细节 3：构造训练矩阵 G 与 X（你问过：G 是什么，X 是什么）

### 8.1 特征矩阵 $G\in\mathbb{R}^{P\times d}$
- $P$ 是训练中出现的扰动条件数量（单基因扰动就是扰动基因种类数）
- 第 $p$ 行是扰动 $p$ 的 embedding（已归一化）：

$$
G_{p,:}=\tilde g(p)
$$

### 8.2 结果矩阵 $X\in\mathbb{R}^{P\times q}$
第 $p$ 行是该扰动条件的输出目标向量：
- 可能是 pseudo-bulk 平均表达 $\bar{X}_p$；
- 也常用平均效应 $\Delta_p=\bar{X}_p-\bar{X}_c$。

$$
X_{p,:}=\bar{X}_p\quad \text{或}\quad X_{p,:}=\bar{X}_p-\bar{X}_c
$$

### 8.3 “是不是学习从 G 到 X？”（你问过）
是的：GenePert 学的是一个映射，使得

$$
G\theta \approx X
$$

---

## 9. Method 细节 4：岭回归与 L2 正则（你问过：岭回归是什么、L2 正则是什么）

### 9.1 多输出线性回归
学习参数矩阵

$$
\theta\in\mathbb{R}^{d\times q}
$$

使得预测为

$$
\hat{X} = G\theta
$$

### 9.2 岭回归（Ridge Regression）
岭回归 = 最小二乘 + L2 正则：

$$
\hat{\theta}=\arg\min_{\theta}\|X-G\theta\|_F^2+\lambda\|\theta\|_F^2
$$

其中：
- $\|\cdot\|_F$ 是 Frobenius 范数（矩阵元素平方和再开方；这里通常用平方形式）。
- $\lambda>0$ 控制正则强度。

### 9.3 L2 正则在“直觉上”做了什么
- 惩罚权重过大，让模型更“保守/平滑”；
- 减少过拟合，提升未见扰动时的外推稳定性。

---

## 10. 未见扰动（unseen perturbations）GenePert 如何应对（你问过）
关键逻辑：**未见基因也能生成输入特征**，然后用已学映射输出预测。

对一个训练中没出现过的基因 $\tilde p$：
1) 仍可从文本得到 $g_{\text{GenePT}}(\tilde p)$，再归一化得到 $\tilde g(\tilde p)$；
2) 预测：

$$
\hat{x}(\tilde p)=\tilde g(\tilde p)^\top \hat{\theta}\in\mathbb{R}^q
$$

因此你说的“没见过这个扰动也能通过 GenePert 生成”，准确含义是：
- **生成的是预测的表达效应向量**（不是生成新的模型参数）。

---

## 11. 双基因扰动怎么做（Method 扩展）
对双扰动 $(p_1,p_2)$，用相加再归一化得到输入特征：

$$
\tilde g(p_1,p_2)=\frac{g(p_1)+g(p_2)}{\|g(p_1)+g(p_2)\|_2}
$$

特点：
- 置换不变：$p_1+p_2=p_2+p_1$；
- 只要两个基因都有 embedding，就能预测未见组合。

---

## 12. 评估指标（你见过式 (4)）：RMSE 与 Pearson（相对对照效应）
评估常在“relative scale”上比较：
- 预测效应：$\hat{\Delta}_p=\tilde X_p-\bar X_c$
- 真实效应：$\Delta_p=\bar X_p-\bar X_c$

相关性（每个扰动算 PCC，再对扰动平均）：

$$
\text{Correlation}=\frac{1}{P}\sum_{p=1}^P \text{PCC}(\hat{\Delta}_p,\Delta_p)
$$

---

## 13. 偏差 vs 方差（你问过：区别、标志是什么）
- 偏差大（欠拟合）：训练误差高、验证误差也高，差距不大。  
- 方差大（过拟合）：训练误差低、验证误差高，train–val gap 大，结果对随机性敏感。  

GenePert 的立场：在高维稀疏 + 扰动种类有限场景，复杂深模易方差大；ridge + 强表征更稳。

---

## 14. ESM2 是什么（你问过）以及与 GenePT 的关系
- ESM2：蛋白质语言模型（输入氨基酸序列，输出 embedding）。
- 在本文里，ESM2 embedding 常作为替代表征或与 GenePT 拼接，测试“序列信息 + 文本语义”是否互补。

你提的类比：“像 RDKit2D 吗？”
- 相似：都是先把对象 featurize 成向量，再用简单模型预测；新对象也能算特征（冷启动友好）。
- 不同：RDKit2D 是确定性理化描述符；GenePT 是知识语义向量。

---

## 15. Table 1 vs Table 2（你问过）
- Table 1：主 benchmark，对比不同 embedding/方法（GenePT、ESM2、Geneformer、scGPT、均值基线等）。
- Table 2：围绕 GenePert 的消融/融合（基因名 embedding、GenePT+ESM2 拼接）。

---

## 16. 与 DeShift 的关系（你问过：能否用 GenePert 替代 DeShift embedding 再做 OT？）

### 16.1 层级差异（必须分清）
- GenePert 的 embedding：$g_{\text{GenePT}}(p)$ 是 **gene-level（扰动条件级）**。
- DeShift 的 VAE embedding：$z_i$ 是 **cell-level（每个细胞一个 latent）**，对照与扰动细胞都有。

因此：
- 不能用 gene-level GenePT 直接替代 cell-level latent 去做 OT（OT 对齐的是细胞分布）。

### 16.2 你确认过的“路线 A”（推荐）
保留 VAE 的 cell latent 与 cell-level OT，仅替换/增强扰动条件表示：
- 原 perturbation embedding（one-hot/learnable）替换为 GenePT embedding（再投影到模型需要的维度）。
- 注入方式：concat / FiLM / conditional LN / additive shift。

### 16.3 关于“Residual head 输入 p 能否从 OT 差值改为 GenePT？”
要点：
- OT 差值（或 OT 推断的位移）是 **数据驱动的分布几何信号**（cell-level）。
- GenePT 是 **扰动语义信号**（condition-level）。
更合理的是融合而非完全替换：
- 例如 residual head 输入同时包含 $z_i$、$\Delta z_{\text{OT}}$、以及 GenePT 条件 $\tilde e_p$。

---

## 17. 快速复现思路（从 AnnData 到 GenePert）
1) 从 `adata` 中得到表达矩阵（确保与论文一致：normalize total=1e4 + log1p）  
2) 选 HVGs：$q=5000$  
3) 按扰动条件分组，算 pseudo-bulk：$\bar X_p$ 与 $\bar X_c$  
4) 构造 $X$（用 $\bar X_p$ 或 $\bar X_p-\bar X_c$）  
5) 准备每个扰动基因的 GenePT embedding，构造 $G$（每行 L2 归一化）  
6) 用岭回归拟合 $\theta$  
7) 对未见基因 $\tilde p$：算 $\tilde g(\tilde p)$，输出 $\tilde g(\tilde p)^\top\hat{\theta}$

---

## 18. 你需要牢记的两句“防混淆”
1) GenePert 学的是 **从扰动基因 embedding 到表达效应**，不是学细胞 embedding。  
2) DeShift 的 OT 对齐对象是 **细胞 latent 分布**；GenePT 更适合用于 **扰动条件表示**，提升未见扰动泛化。
