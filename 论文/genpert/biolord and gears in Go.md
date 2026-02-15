---
title: "GEARS 与 bioLORD 如何利用 GO 图（实现细节版）"
tags: [scRNA-seq, perturbation, GEARS, bioLORD, GO, GNN]
aliases: ["GO图在GEARS/bioLORD中的实现", "GEARS GO graph implementation"]
---

## 说明
本笔记整理自公开论文与综述描述，重点是**实现层面的流程与模块**。  
如果需要更精确的公式/超参数，请以原论文 Methods 与官方代码为准。

参考链接见文末。

---

## 1) GEARS 如何使用 GO 图（实现细节）

### 1.1 双图先验：共表达图 + GO 图
GEARS 同时使用两类图先验，并用 GNN 分别聚合：  
- **基因共表达图（gene coexpression graph）**：用于学习 **gene embeddings**。  
  - 做法：对训练集基因表达计算 Pearson 相关系数 ρ，  
    为每个基因连接“相关性最高且超过阈值 δ”的前 H 个基因。  
  - 对该图运行 GNN，得到 gene embedding。  
  参考：GEARS 原论文方法段。  
  https://www.nature.com/articles/s41587-023-01905-6  
- **GO 派生图（gene perturbation similarity graph）**：用于学习 **perturbation embeddings**。  
  参考：GEARS 原论文方法段。  
  https://www.nature.com/articles/s41587-023-01905-6  

### 1.2 GO 图如何构建（实现细节）
GEARS 先构建 GO 二分图，再得到扰动相似图：  
1. 定义 **基因–GO term 二分图**。  
2. 对任意基因对 (u, v) 计算 **Jaccard 指数**：  
   - J(u,v) = |N_u ∩ N_v| / |N_u ∪ N_v|  
3. 对每个基因 u 选择 J(u,v) 最高的前 H 个基因，  
   形成 **gene perturbation similarity graph**。  
4. 在该图上用 GNN 更新 perturbation embedding。  
参考：GEARS 原论文方法段。  
https://www.nature.com/articles/s41587-023-01905-6  

### 1.3 预测管线（结构级实现）
GEARS 的预测流程（简化版）：  
1. **GNN(共表达图)** -> gene embedding  
2. **GNN(GO 图)** -> perturbation embedding  
3. 对多基因扰动：对 perturbation embeddings 做 **sum 聚合 + MLP**  
4. 将 perturbation 表征与 gene embedding 组合，得到 post‑perturb gene embedding  
5. **cross‑gene decoder/MLP** 汇总全基因信息  
6. **gene‑specific linear layer** 生成每个基因的扰动效应  
参考：GEARS 原论文方法段。  
https://www.nature.com/articles/s41587-023-01905-6  

### 1.4 为什么能泛化到未见扰动
GO 图提供“功能相似”的结构先验。  
当目标基因未在训练集中出现，但在 GO 图中与已见基因相连时，  
GNN 可以通过邻居传播获得信息，从而实现未见扰动预测。  
https://www.nature.com/articles/s41587-023-01905-6  

---

## 2) bioLORD 如何使用 GO 图（实现细节）

bioLORD 是一个**解耦式生成框架**，在基因扰动任务中使用 GO 图作为“扰动特征/先验”。  
论文在 genetic perturbation 设置中明确说明：  
**使用 GEARS 的 GO 图边作为扰动特征**，并说明该 GO 图通过“共享显著 GO terms 的基因对”建立加权边。  
https://www.nature.com/articles/s41587-023-02079-x  
https://pmc.ncbi.nlm.nih.gov/articles/PMC11554562/  

### 2.1 Perturb-seq（单基因扰动）设置
论文描述：  
1. 使用 GEARS 预处理的 AnnData。  
2. **用 GEARS GO 图中的扰动边作为扰动特征**。  
3. 训练时只使用“扰动条件的平均表达”作为目标。  
https://pmc.ncbi.nlm.nih.gov/articles/PMC11554562/  

### 2.2 Perturb-seq（双基因扰动）设置
论文描述：  
1. 同样使用 GEARS 的 GO 图作为扰动特征。  
2. 训练阶段只使用 **单基因扰动** 与对照。  
3. 对双基因扰动，论文中采用“两个单基因预测差异之和”进行近似。  
https://pmc.ncbi.nlm.nih.gov/articles/PMC11554562/  

### 2.3 训练与评估要点
- 采用 GEARS 论文中定义的训练/测试拆分策略。  
- 评估指标包含 normalized MSE 等。  
https://pmc.ncbi.nlm.nih.gov/articles/PMC11554562/  

---

## 3) 与 GO 图相关的常见实现要点（汇总）

- **GO 图提供“扰动先验结构”**，不是直接预测表达。  
- **关键在于 GNN 聚合**：将邻居信息注入扰动 embedding。  
- **泛化能力**来自于“相似基因共享 GO 结构”。  

---

## 参考链接
- GEARS 原论文（Nature Biotechnology, 2023）  
  https://www.nature.com/articles/s41587-023-01905-6  
- GEARS PMC 版本（开放全文）  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC11180609/  
- bioLORD 原论文（Nature Biotechnology, 2023）  
  https://www.nature.com/articles/s41587-023-02079-x  
- bioLORD PMC 版本（开放全文）  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC11554562/  
