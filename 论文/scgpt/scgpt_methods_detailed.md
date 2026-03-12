# scGPT 论文详细讲解（三）
## 对应部分：4 Methods

> 这一份专门讲 Methods。  
> 阅读目标不是记公式，而是搞清楚：**每个模块是为了解决什么单细胞建模难点，模块之间如何拼起来，最终为什么能支持多任务迁移。**

---

# 0. Methods 总览：scGPT 到底在做什么？

从方法上看，scGPT 可以概括为一句话：

> 把单细胞数据转写成“基因 token + 表达值 + 条件信息”的上下文序列，再用经过改造的 Transformer 做生成式预训练，并在此基础上迁移到多种单细胞任务。

Methods 的核心逻辑分成六层：

1. **数据表示**：单细胞矩阵如何 token 化；
2. **输入嵌入**：gene / value / condition 如何合并；
3. **Transformer 主干**：如何做上下文建模；
4. **生成式预训练目标**：模型到底学什么；
5. **任务适配**：如何迁移到 annotation / integration / perturbation 等；
6. **表示与解释**：如何抽出 cell embedding 与 gene embedding。

---

# 1. 输入表示：为什么单细胞数据能变成 GPT 输入？

## 1.1 原始输入是什么？

单细胞数据最基本的形式，是一个 **cell × gene matrix**。  
矩阵里的元素表示某个细胞中某个基因的表达量；在其它模态里，也可以是 peak 可及性、蛋白信号等。

但 Transformer 不直接吃这种“匿名的大矩阵”。  
它更适合吃的是一组有身份的元素序列。

所以 scGPT 做的第一步，就是把矩阵的一行（一个细胞）改写成模型可消费的结构化输入。

---

## 1.2 三类核心输入组件

论文中，scGPT 的输入由三层信息组成：

### （1）Gene tokens
每个基因都被看成一个离散 token，并映射到唯一 ID。  
这一步对应 NLP 里的“词表”。

这一设计的意义在于：
- 模型知道“这个位置对应的是哪个基因”；
- 不同细胞里同一个基因共享同一个身份嵌入；
- 模型可以逐渐学到 gene-level semantics。

---

### （2）Expression values
只有基因身份不够，因为同一个基因在不同细胞里的表达强度不同。  
因此 scGPT 还要输入表达值信息。

但这里有一个单细胞特有难点：

> 表达值是连续且高度稀疏、跨批次尺度不稳定的，不像 NLP token 那样天然离散。

为了解决这个问题，scGPT 没有直接把原始 count 当作稳定语义，而是引入了 **value binning**。

---

### （3）Condition tokens
这部分特别关键。  
单细胞数据往往伴随着大量上下文：
- batch
- modality
- perturbation condition
- 其它任务相关标签或元信息

scGPT 把这些上下文编码成额外的 condition tokens / embeddings。  
这样模型在解释表达模式时，不是脱离背景地看数值，而是在“当前条件”下理解该细胞。

---

## 1.3 为什么这三者缺一不可？

- 只有 gene token，没有 value：模型只知道“谁在场”，不知道“表达多高”；
- 只有 value，没有 gene token：模型只看到数值，不知道这个数值对应谁；
- 没有 condition token：模型会把 batch / modality / perturbation 混入内容本身。

所以 scGPT 的输入设计本质上是在统一三类信息：
**内容身份 + 强度大小 + 背景条件。**

---

# 2. Value binning：为什么这是方法里的关键小设计？

## 2.1 问题背景

单细胞表达值有几个著名问题：

1. **不同实验批次的尺度差异很大**；
2. **零值很多**，存在 dropout；
3. **绝对 count 的可比性差**；
4. 下游更关心相对高低和上下文组合，而不一定是精确 count。

如果直接把原始表达值喂给模型，模型很容易过拟合技术尺度差异。

---

## 2.2 scGPT 的做法

scGPT 使用 **cell-specific value binning** 的思想。  
直观地说，就是把每个细胞内部非零表达值按相对大小离散到若干区间中。

这样做有几个效果：

### 效果一：把连续噪声较大的数值转成更稳健的相对等级
模型更容易学“高 / 中 / 低表达”的关系，而不是被精确 count 干扰。

### 效果二：缓解跨批次尺度不一致
不同实验中的绝对值可能不可比，但相对等级更稳定。

### 效果三：让表达值更适配 embedding 化
离散 bin 很适合像 token 一样被嵌入到向量空间。

---

## 2.3 为什么这不是一个无关紧要的预处理技巧？

因为它直接决定了模型学到的“值语义”是什么。

如果不用 binning，模型可能学到：
- 某个平台 count 偏大；
- 某批次测序更深；
- 某些技术偏差。

而用了 binning，模型更可能学到：
- 一个细胞内哪些基因相对突出；
- 某种状态下哪些基因通常共同位于高表达层级；
- 值模式和细胞身份之间的关系。

所以 value binning 实际上是把单细胞表达从“绝对测量”改造成更适合 foundation model 的“相对语义信号”。

---

# 3. 输入嵌入：三类信息如何合并？

## 3.1 基本思想

对于每个位置，scGPT 会把：
- gene identity embedding
- expression/value embedding
- condition embedding

合成一个统一的输入向量。

你可以把它理解成：

> “某个基因在当前细胞、当前条件下，以某种表达强度出现”  
> 这一整件事，被压成一个 token-level representation。

---

## 3.2 这种合并有什么好处？

### 好处一：上下文从输入阶段就被显式注入
batch / modality / perturbation 不再是事后补丁，而是模型从第一层开始就能利用的信息。

### 好处二：同一个基因在不同条件下可以有不同上下文表示
这对 perturbation / multi-omics 非常重要。

### 好处三：不同任务可以共享同一输入接口
无论做 annotation、integration 还是 perturbation，都可以复用这套输入表达范式。

---

# 4. Transformer 主干：为什么选择 Transformer？

## 4.1 核心原因

单细胞数据最难的地方之一，是**高维基因之间存在复杂非线性依赖**。  
一个基因的解释，不是看它自己，而是看它与许多其它基因共同出现的上下文。

Transformer 的优势在于：
- 能建模全局依赖；
- 多头注意力可捕捉不同关系子空间；
- 适合作为大规模预训练主干；
- 迁移到不同任务时接口统一。

---

## 4.2 但单细胞和 NLP 的关键差异是什么？

最大的差异是：  
**基因集合没有天然语序。**

自然语言句子有 token 顺序，左到右预测有明确语义；  
单细胞基因集合没有这种线性顺序。

这意味着 scGPT 不能直接照搬标准自回归 GPT。  
否则“前面基因预测后面基因”这个顺序本身就没有生物意义。

这就是为什么 Methods 中 attention mask 的设计如此重要。

---

# 5. 生成式预训练：scGPT 到底在学什么？

## 5.1 目标不是简单重构整行表达矩阵

如果只是普通 autoencoder 式重构，模型很可能学到：
- 平滑表示；
- 常见基因均值；
- 技术噪声模式。

scGPT 更想学的是：

> 在给定部分已知基因表达和条件上下文时，预测未知部分基因表达。

这使得训练目标从“无条件复制输入”变成了“有上下文的条件生成”。

---

## 5.2 特殊 attention mask 的意义

由于没有天然语序，作者设计了专门 attention masking 机制。  
核心目的不是模仿 NLP 里的 causal mask，而是人为规定一类“已知/未知”关系，使模型在训练时始终处于：

- 看见一部分基因；
- 推断另一部分基因；

这样的条件生成状态。

这一步非常关键，因为它把预训练任务变成了：

### “基因上下文补全”
模型必须利用已知基因和 condition token，去推断其它基因的合理值。

于是模型被迫学习：
- 基因共表达模式；
- 细胞状态程序；
- 条件变化下表达如何协同改变。

---

## 5.3 为什么这是 foundation model 所需要的目标？

因为 foundation model 需要学共性结构。  
而“根据部分上下文恢复整体”正是一种非常强的结构学习方式。

它不像纯监督分类那样只围绕标签学；
也不像简单重构那样容易学到输入复制。  
它更像在逼模型掌握“一个细胞内部的组织规律”。

---

# 6. 预训练任务与辅助目标

> 论文和官方实现中，除了主生成目标外，还配合若干任务相关目标来增强表示质量。

---

## 6.1 Gene Expression Prediction（GEP / 主生成思路）

这是最核心的目标。  
模型根据可见上下文预测被遮蔽或待生成的基因表达。

它让模型学到：
- 哪些基因倾向共同激活；
- 某种状态下缺失部分表达时整体应该怎样补全；
- 条件变化时表达模式的迁移规律。

---

## 6.2 Masked Value / Gene Expression Prediction for Cell Embedding（GEPC / MVC 类目标）

这类目标的直觉是：  
不仅要让模型会预测单个值，还要让 **cell embedding 本身** 对基因值预测有帮助。

它的作用可以理解为：
- 逼 cell-level representation 压缩更丰富的信息；
- 让 CLS / pooled cell representation 不只是下游分类用，而是对表达恢复本身也有约束。

这会提高细胞嵌入的“信息密度”。

---

## 6.3 Elastic Cell Similarity（ECS）等任务相关正则

在公开教程和实现中，scGPT 还使用了一些增强表征质量的目标，例如 ECS。  
这类目标通常是为了：
- 让同类细胞在嵌入空间更稳定；
- 提高 integration / annotation 场景下的 embedding 质量；
- 减少表征空间塌缩或过度分散。

在理解上，你可以把它们看成：
**主生成目标之外，为 cell representation 加的结构化约束。**

---

# 7. Cell embedding：模型最终如何得到细胞表示？

## 7.1 为什么需要专门 cell embedding？

因为很多下游任务最终不是逐基因输出，而是需要一个细胞级 summary：
- annotation
- clustering
- batch integration
- reference mapping

所以 Transformer 输出之后，必须能提取一个稳定的细胞向量表示。

---

## 7.2 这个表示为什么重要？

因为它相当于 scGPT 压缩后的“细胞状态语义向量”。  
如果这个向量足够好，那么很多任务只需要在上面接一个轻量头部。

也就是说，预训练的价值很大程度上体现在：
**cell embedding 是否已经包含足够多的可迁移信息。**

---

# 8. Task-specific adaptation：同一个 backbone 如何适配不同任务？

这部分是 Methods 的另一个关键点。  
foundation model 不是只会预训练，还要会迁移。

---

## 8.1 细胞类型注释

做法上可以理解为：

- 用预训练 backbone 提取 cell embedding；
- 在其上接分类头；
- 用有标签数据微调。

这里的关键不是分类头多复杂，而是 backbone 已经学到了足够好的细胞表示，所以微调效率更高、性能更稳。

---

## 8.2 Batch integration

在 integration 场景里，目标不再是分类，而是让不同批次的同类细胞在嵌入空间靠近。  
因此会配合：
- batch/domain 信息输入；
- embedding 层面的对齐目标；
- 生物保持与批次去除之间的平衡约束。

scGPT 的优势在于，batch 本身被纳入 condition token，  
模型从输入阶段就知道“哪些差异可能是技术差异”。

---

## 8.3 Multi-omic integration

对于多组学任务，关键不是强行让不同模态完全同分布，  
而是让模型在共享 backbone 中学习：
- 模态共享语义；
- 模态特异上下文；
- 同一细胞跨模态的一致性。

因此 modality token / condition token 在这里扮演了“显式上下文桥梁”的角色。

---

## 8.4 Perturbation prediction

这是 Methods 中最值得单独注意的一块。  
其输入不只是初始表达，还包括 perturbation 条件。  
模型需要学的是：

> 在给定初始 cell state 和 perturbation token 时，生成扰动后的表达。

从方法角度看，这一步说明 scGPT 的 backbone 不是只服务静态表示学习，  
而是可以自然扩展到**条件状态转移建模**。

对你做药物扰动任务来说，这一点尤其重要。  
因为它意味着 scGPT 的核心能力可以被理解为：

- 状态编码；
- 条件注入；
- 条件下的表达生成。

这和很多 perturbation model 的基本框架是相通的。

---

# 9. Gene embedding 与 gene network inference：方法上怎么实现？

## 9.1 Gene embedding 从哪里来？

由于每个基因都有 token embedding，并在预训练中反复参与上下文建模，  
模型会逐步形成一个 gene representation space。

这个空间不是手工定义的，而是从大规模细胞上下文中学出来的。

---

## 9.2 为什么它能支持 gene program / network 分析？

因为如果模型真的学到了：
- 哪些基因经常在相似状态中共同出现；
- 哪些基因在相近上下文中扮演相似功能；
- 哪些基因的依赖结构经常被注意力捕捉；

那么 embedding 距离、图结构、attention pattern 就可能成为：
- gene module 提取的基础；
- interaction 候选关系的来源；
- program activation 分析的先验。

这里的方法论重点不是“模型直接输出 GRN”，  
而是“模型学出的内部结构可以被二次分析为网络候选”。

---

# 10. Methods 的真正核心：统一接口 + 条件生成

如果把整个 Methods 压成一句话，我会这样总结：

> scGPT 的方法核心，不是简单把 Transformer 用到单细胞上，而是围绕“无自然顺序、数值噪声强、上下文复杂、任务多样”这几个单细胞特有问题，设计了一套统一输入接口、条件感知表示和生成式预训练机制。

这套方法为什么强？因为它同时解决了：

- **表示问题**：怎么把单细胞矩阵变成可预训练输入；
- **噪声问题**：怎么让表达值更稳健；
- **上下文问题**：怎么把 batch / modality / perturbation 纳入模型；
- **学习问题**：怎么在无自然语序的数据上做生成式预训练；
- **迁移问题**：怎么用同一个 backbone 支持多任务。

---

# 11. 你做研究时最值得借鉴的 Methods 思想

## 思想一：输入设计决定上限
很多人只关注 backbone，但 scGPT 告诉你，  
真正决定模型能否统一多任务的，首先是输入表示是否把 gene / value / condition 三者理顺。

## 思想二：预训练目标要贴近“结构学习”
不要只做简单重构。  
更有价值的是让模型根据部分上下文恢复整体，这样才能逼它学到共性程序。

## 思想三：condition 建模是单细胞生成任务的核心
batch、modality、perturbation、time 等条件，不是附属信息，而是决定表达语义的重要变量。

## 思想四：foundation model 不只输出 cell embedding
gene embedding、attention structure、program-level pattern 同样重要。  
这些内容常常决定模型是否能用于机制分析。

---

# 12. 一句话总结 Methods

> scGPT 的 Methods 本质上是在为单细胞数据重新发明一套“GPT 语言”：用 gene token 表示身份，用 value binning 表示相对强度，用 condition token 表示上下文，再通过特制的注意力掩码和生成式预训练目标学习细胞内部的基因协同结构，最终把这套知识迁移到注释、整合、扰动预测和基因网络分析等任务上。

