# TriShift：面向单细胞扰动预测的参考条件化状态转移建模

中文论文初稿

## 1. Abstract

单细胞扰动预测旨在利用有限实验观测，推断未测扰动条件下的转录组响应。现有方法通常将该任务近似为从扰动条件到表达谱的直接映射，但这一表述容易弱化控制细胞状态在预测中的作用，也难以区分模型究竟是在恢复扰动特异信号，还是仅在拟合稳定存在的系统性偏移。本文提出 TriShift，将单细胞扰动预测重新表述为参考条件化（reference-conditioned）的状态转移问题。TriShift 首先使用去噪变分自编码器学习可比较的潜空间；随后在潜空间中执行控制细胞检索，并利用扰动语义表示学习条件化潜在位移；最后通过表达生成器整合控制状态、扰动表示与位移信息，输出扰动后表达预测。与直接的 condition-to-expression 映射不同，TriShift 将参考匹配、位移建模与表达生成组织为统一链路。围绕这一方法主线，本文设计了覆盖五个单扰动数据集的统一实验框架，并将 Norman 组合扰动作为细粒度 hard-generalization 压力测试。作为当前中文底稿，本文重点给出问题定义、方法公式化描述、实验协议与指标说明；定量结果与正式文献引用将在实验完成后进一步补充。

## 2. Introduction

大规模单细胞扰动实验正在持续推动功能基因组学、药物机制研究与基因调控建模的发展，但实验上可被真实测量的扰动空间仍远小于理论可能空间。因此，如何利用已有扰动数据预测未测条件下的单细胞表达响应，已成为单细胞机器学习中的核心问题之一。[文献待补：单细胞扰动预测综述与基准] 近年来的主流方法大致沿两条路径展开：一类方法把扰动条件编码为离散标签、图结构或知识先验，再直接回归扰动后表达；另一类方法引入语义嵌入或基础模型表示，试图提升对未见基因或未见条件的外推能力。[文献待补：GEARS, CPA, biolord, Scouter, scGPT] 尽管这些方法在多个基准上取得了显著进展，但它们多数仍默认扰动条件足以决定目标表达，而将控制细胞状态视为弱条件或背景信息。

这一默认假设在单细胞场景中往往过强。相同扰动施加于不同控制细胞亚群时，转录组响应的方向、幅值与细胞异质性均可能不同；若模型没有显式表示“从哪个参考状态出发”，则预测过程容易退化为对条件平均效应的拟合，而难以准确恢复 state-dependent 的扰动响应。更重要的是，近期评估工作指出，许多常用指标容易受到系统性偏移（systematic variation）的影响。所谓系统性偏移，是指在预测样本与真实样本之间稳定存在、但并不等同于 perturbation-specific signal 的整体差异，例如控制池选择偏差、群体层面基础状态差异或全局表达漂移。若评估仅依赖静态均值相关性或误差，模型可能因为捕捉到这类公共偏移而获得偏乐观分数。[文献待补：Systema]

Systema 的一个核心启发是，单细胞扰动预测的评估应尽量超越 systematic variation，转而考察模型是否真正恢复了相对于参考控制状态的扰动增量。[文献待补：Systema] 这一观点不仅影响评估设计，也反过来影响任务建模：如果真正重要的是“相对于控制状态的变化”，那么方法本身就不应被表述为单纯的 condition-to-expression 回归，而应被表述为参考条件化的状态转移过程。

基于这一认识，本文提出 TriShift。TriShift 的核心观点是，扰动响应可以理解为控制状态在潜空间中的条件化位移，而非脱离参考状态的静态表达终点。为此，模型首先学习稳定且可比较的单细胞潜空间；随后在潜空间中执行 reference-conditioned 检索，获得与目标扰动细胞更一致的控制参考；再结合控制状态表示与扰动语义表示，预测潜在位移；最后将位移重新映射回表达空间。这样的建模方式使参考匹配不再只是预处理技巧，而成为方法定义的一部分。

本文的主要贡献体现在四个方面。第一，我们将单细胞扰动预测正式表述为参考条件化状态转移问题，而非直接的条件回归。第二，我们构建了由 `state`、`genept` 与 `latent_mu` 组成的分工式表示体系，其中 `genept` 提供扰动语义，`latent_mu` 负责潜空间匹配与位移监督，`state` 负责向生成器提供控制状态。第三，我们在方法中显式引入最优传输（optimal transport, OT）风格的控制参考匹配，以在群体层面构造更合适的控制原型。第四，我们围绕主文图顺序组织实验，将跨数据集 benchmark、reference-conditioned 证据、Norman hard-generalization 与 pathway 恢复串联为统一叙事，并在文末 Supplement 中系统解释本文使用的评估指标。

## 3. Related Work

### 3.1 扰动表示与表达生成模型

单细胞扰动预测方法首先在“如何表示扰动条件”这一问题上分化。较早的方法通常将扰动条件视为离散标签，并直接学习从条件到表达的映射；这类方法实现直接，但对未见条件的泛化能力高度依赖训练覆盖。另一类方法则引入图结构、基因网络或知识图谱，通过 cross-gene interaction 建模扰动在基因层面的传播。GEARS 是这一方向的代表，其方法叙事先定义基因表示和扰动表示，再说明二者如何通过结构模块交互，最后落到 gene-specific 解码器。[文献待补：GEARS] 这类方法的优势在于结构归纳偏置清晰，但对图覆盖范围与任务定义较为敏感。

与之相对，Scouter 所代表的语义嵌入路线更强调 categorical extrapolation。该类方法通过预训练文本模型为基因或扰动构造连续向量，再将这些向量与控制表达编码共同输入生成器。其关键直觉是：若未见扰动能够在语义空间中与已见扰动建立邻近关系，模型便有机会借助这一连续几何结构进行更平滑的外推。[文献待补：Scouter] TriShift 延续了这一思路，采用 `genept` 作为扰动语义表示；但与将语义嵌入直接送入解码器不同，TriShift 将其置于参考条件化位移建模链路中。

### 3.2 参考匹配、状态对齐与最优传输

控制细胞如何进入模型，是扰动预测中另一个关键但常被弱化的问题。许多方法把控制细胞仅当作输入背景或样本配对对象，而较少将“参考状态选择”提升为显式建模对象。事实上，若扰动响应本质上是相对于参考状态的变化，那么控制参考的质量将直接影响后续生成结果。scPRAM 提供了重要启发：该方法将变分表示学习与最优传输结合，用于对齐扰动前后的单细胞状态，并强调细胞分布层面的对应关系，而不是仅比较条件均值。[文献待补：scPRAM]

最优传输为控制匹配提供了自然框架。它不要求控制池与扰动池之间存在严格的一一配对，而是允许在边缘约束下学习软耦合关系，从而在群体层面构造更稳定的参考对应。对于单细胞扰动任务而言，这一点尤其重要，因为不同条件下的细胞分布往往包含噪声、密度差异和多模态结构。TriShift 在潜空间 top-k 匹配中引入 OT，正是为了把控制参考构造从“随机抽取”提升为“受潜空间几何约束的群体对齐”。

### 3.3 超越系统性偏移的评估视角

除了建模方式，评估框架也在发生变化。传统指标常关注真实与预测表达的整体相关性或误差，但这类指标不一定能够区分模型是在恢复 perturbation-specific signal，还是仅在拟合公共偏移。Systema 强调，评估应尽量围绕参考差分展开，以识别模型是否真正重建了扰动景观（perturbation landscape）。这一思路与本文的方法主线高度一致。

基于这一评估视角，本文在常规 DE 相关指标之外，引入 `systema_Pearson` 来衡量模型在扣除参考向量后对真实扰动增量的恢复程度；同时保留 Wasserstein 指标和 scPRAM 兼容指标，以分别衡量分布拟合和细胞异质性恢复。这样，TriShift 的方法设计与评估标准能够围绕同一个问题展开：模型是否真正超越系统性偏移，并恢复了相对于控制状态的扰动变化。

## 4. Method

### 4.1 Problem formulation and motivation

设基因表达空间为 $\mathcal{X} \subset \mathbb{R}^{G}$，其中 $G$ 为基因维度。对于每个扰动条件 $c \in \mathcal{C}$，我们观测控制细胞集合 $\{\mathbf{x}^{(0)}_j\}_{j=1}^{N_0}$ 与扰动细胞集合 $\{\mathbf{x}^{(p)}_i\}_{i=1}^{N_p}$。记组成该扰动的基因集合为 $\mathcal{G}(c)$。TriShift 使用预训练基因嵌入构造条件向量

$$
\mathbf{c} = \mathrm{Pool}\big(\{\mathbf{e}_g\}_{g \in \mathcal{G}(c)}\big), \qquad \mathrm{Pool} \in \{\mathrm{sum}, \mathrm{mean}\},
$$

其中 $\mathbf{e}_g \in \mathbb{R}^{d_c}$ 为基因 $g$ 的嵌入向量。本文后续将该条件向量记为 `genept`。这一表述与 Scouter 中“先定义基因语义表示，再解释它如何进入生成模型”的写法一致，但 TriShift 不将 $\mathbf{c}$ 视为表达预测的唯一条件，而将其与参考控制状态共同建模。

本文的目标不是直接学习静态映射 $f(c) \rightarrow \hat{\mathbf{x}}^{(p)}$，而是学习参考条件化状态转移：

$$
\hat{\mathbf{x}}^{(p)} = G\big(\mathbf{x}^{(0)}, \mathbf{c}, \Delta(\mathbf{u}_{\mathrm{ctrl}}, \mathbf{c})\big),
$$

其中 $\mathbf{u}_{\mathrm{ctrl}}$ 是控制状态的低维表征，$\Delta(\cdot)$ 表示在给定扰动条件下诱导的潜在位移，$G(\cdot)$ 表示表达生成器。由此，问题的核心从“扰动标签决定表达终点”转变为“参考控制状态与扰动语义共同决定状态如何迁移”。

直接的 condition-to-expression 映射隐含三个不够稳健的假设。第一，它默认扰动标签本身足以决定目标表达，而忽略了控制状态依赖性。对于单细胞任务，这一假设往往不成立：同一扰动施加于不同控制细胞群体时，响应幅值、方向和细胞异质性均可能不同。若模型没有显式表示“从哪个状态出发”，便容易将预测退化为对条件平均效应的回归。第二，它容易与系统性偏移混淆。按照 Systema 的观点，预测结果即便没有正确恢复 perturbation-specific signal，也可能因为拟合了稳定的公共差异而在整体相关性上表现良好。第三，直接映射难以自然吸收 reference information。若控制参考只在训练前被随机抽取或仅用于数据配对，那么匹配质量不会进入模型主链路，也难以解释 nearest retrieval、random retrieval 与 reference baseline 之间的性能差异。基于这些原因，TriShift 将参考匹配显式纳入方法定义，并把任务改写为参考条件化状态转移。

### 4.2 TriShift overview and representations

TriShift 的核心直觉是，扰动响应可以被理解为控制状态在潜空间中的条件化位移。模型首先需要确定“从哪个控制状态出发”，随后才需要建模“扰动会把该状态推向哪里”。这一定义使控制匹配、位移建模和表达生成形成递进链路，而非平行堆叠输入。本文主文默认实例以 [adamson config](E:/CODE/trishift/scripts/trishift/adamson/config.yaml) 和 [norman config](E:/CODE/trishift/scripts/trishift/norman/config.yaml) 的共同设置为准：使用 OT 匹配、joint 训练、`latent_mu` 作为 shift 输入、sum pooling 的 `genept`、transformer-based shift 模块、`full` generator 输入模式以及 residual head。形式上，TriShift 的信息流可写为

$$
\mathbf{x}^{(0)} \xrightarrow{\text{encoder}} \{\mathbf{z}_{\mathrm{state}}, \mathbf{z}_{\mu}\}
\xrightarrow[\mathbf{c}]{\text{matching + shift}} \mathbf{h}
\xrightarrow{\text{generator}} \hat{\mathbf{x}}^{(p)},
$$

其中 $\mathbf{z}_{\mu}$ 用于检索与潜在位移监督，$\mathbf{z}_{\mathrm{state}}$ 用于表达生成，$\mathbf{h}$ 表示 shift 模块输出的紧凑位移表征。TriShift 由三个阶段组成。Stage 1 使用去噪变分自编码器学习可比较的潜空间。Stage 2 在潜空间中执行控制参考匹配，并以控制状态和扰动语义为输入学习 shift 表征。Stage 3 则将控制状态、扰动语义和 shift 信息映射回表达空间。这样的结构并非将所有输入简单拼接，而是先建立稳定状态坐标，再学习位移，最后生成表达。其他未启用的结构变体，例如 MLP shift 模块或 cross-attention 版本，不作为本文主文默认实例的一部分，可在后续补充实验中说明。

#### 4.2.1 `state`

Stage 3 的压缩器 $f_{\mathrm{enc}}$ 将控制表达 $\mathbf{x}^{(0)}$ 映射为低维状态表示

$$
\mathbf{z}_{\mathrm{state}} = f_{\mathrm{enc}}(\mathbf{x}^{(0)}) \in \mathbb{R}^{d_s}.
$$

`state` 主要进入生成器，用于保留当前细胞所处的控制状态。它回答的是“当前细胞是什么状态”，而不是“扰动如何改变该状态”。

#### 4.2.2 `genept`

`genept` 对应上文的条件向量 $\mathbf{c}$。其构造方式与 Scouter 中的 GenePT 思路一致，即先为每个基因构造连续语义向量，再对多基因扰动做池化：

$$
\mathbf{c} = \mathrm{Pool}\big(\{\mathbf{e}_g\}_{g \in \mathcal{G}(c)}\big).
$$

这一表示的意义不在于替代控制状态，而在于为未见基因和未见条件提供连续语义结构，从而缓解 purely categorical 表示带来的外推困难。

#### 4.2.3 `latent_mu`

Stage 1 编码器输出潜均值

$$
\mathbf{z}_{\mu} = \boldsymbol{\mu}_{\phi}(\mathbf{x}) \in \mathbb{R}^{d_z}.
$$

`latent_mu` 的主要职责是提供可比较、可检索的潜空间坐标，用于 top-k 匹配、控制原型构造与潜在位移监督。与 `state` 相比，它更强调跨细胞比较时的几何可比性；与 `genept` 相比，它不编码扰动语义，而编码控制状态的位置。

#### 4.2.4 How representations interact

TriShift 不将 `state`、`genept` 与 `latent_mu` 视为可互换的三个向量，而是赋予它们不同职责。首先，`latent_mu` 用于参考匹配和潜在 delta 目标构造；其次，`genept` 与控制侧主向量共同进入 shift 模块；最后，`state` 在生成器处与 shift 结果汇合，恢复目标表达。对于主文默认实例，生成器采用 `full` 输入模式：

$$
\mathbf{g}_{\mathrm{in}}=
[\mathbf{c}; \mathbf{z}_{\mathrm{state}}; \mathbf{h}],
$$

并由

$$
\hat{\mathbf{x}}^{(p)} = f_{\mathrm{gen}}(\mathbf{g}_{\mathrm{in}})
$$

输出预测表达。因此，更合适的概念表达不是静态的 `state || genept || latent_mu`，而是：`latent_mu` 负责匹配与监督，`genept` 负责扰动语义，`state` 负责控制状态，三者通过 shift 与 generator 构成分阶段信息流。其他 generator 输入模式仅作为实现变体存在，不作为主文默认实例的一部分。

### 4.3 Stage 1 latent space learning

给定输入表达 $\mathbf{x}$，首先注入高斯噪声

$$
\tilde{\mathbf{x}} = \mathbf{x} + \boldsymbol{\epsilon}, \qquad \boldsymbol{\epsilon} \sim \mathcal{N}(0, \sigma_n^2 \mathbf{I}),
$$

编码器给出高斯后验

$$
q_{\phi}(\mathbf{z}\mid \tilde{\mathbf{x}})
=
\mathcal{N}\!\big(\boldsymbol{\mu}_{\phi}(\tilde{\mathbf{x}}), \mathrm{diag}(\exp(\log \boldsymbol{\sigma}_{\phi}^2(\tilde{\mathbf{x}})))\big).
$$

经重参数化采样得到 $\mathbf{z}$，并由解码器重构 $\hat{\mathbf{x}}$。Stage 1 的训练目标为

$$
\mathcal{L}_{\mathrm{stage1}}
=
\frac{1}{B}\sum_{b=1}^{B}
\left[
\frac{1}{2}\lVert \mathbf{x}_b - \hat{\mathbf{x}}_b \rVert_2^2
+
\frac{1}{2}\beta \lambda_{\mathrm{kl}}
\mathrm{KL}\!\left(q_{\phi}(\mathbf{z}\mid \tilde{\mathbf{x}}_b)\,\|\,\mathcal{N}(0,\mathbf{I})\right)
\right].
$$

训练结束后，模型缓存所有细胞的潜均值

$$
\mathbf{z}_{\mu} = \boldsymbol{\mu}_{\phi}(\mathbf{x}),
$$

供后续匹配和 delta 监督使用。

### 4.4 Reference matching and shift modeling

Stage 2 的控制侧输入主向量定义为

$$
\mathbf{u}_{\mathrm{ctrl}}=
\begin{cases}
\mathbf{z}_{\mu}^{(0)}, & \texttt{shift\_input\_source}=\texttt{latent\_mu}, \\
\mathbf{z}_{\mathrm{state}}, & \texttt{shift\_input\_source}=\texttt{state}.
\end{cases}
\qquad
\mathbf{h}=f_{\mathrm{shift}}(\mathbf{u}_{\mathrm{ctrl}},\mathbf{c}).
$$

在本文主文默认实例中，Stage 2 采用单层 transformer block，并使用 `concat` 读出作为 shift 表征；由于 `predict_delta=false`，因此 $\mathbf{h}$ 被解释为供生成器使用的紧凑 shift representation，而不是显式的 $\Delta \mathbf{z}$ 预测。当前 Adamson 与 Norman 主配置进一步将该 shift 表征维度固定为 8。

在 Stage 2 之前，TriShift 需要为每个扰动样本构造更合适的控制参考。主文默认实例在潜空间中采用 OT 匹配，并令每个扰动样本使用的 top-k 控制参考数满足

$$
k = \max\big(1,\ \mathrm{round}(0.01\,N_{\mathrm{ctrl}})\big),
$$

其中 $N_{\mathrm{ctrl}}$ 表示当前训练控制池中的细胞数。对于当前 Adamson 与 Norman 主配置，这一规则分别实例化为 `k=300` 与 `k=100`。给定控制潜向量 $\{\mathbf{z}^{(0)}_i\}_{i=1}^{n_0}$ 和扰动潜向量 $\{\mathbf{z}^{(p)}_j\}_{j=1}^{n_p}$，定义代价矩阵

$$
C_{ij} = \lVert \mathbf{z}^{(0)}_i - \mathbf{z}^{(p)}_j \rVert_2.
$$

记 $\mathbf{a}\in\mathbb{R}_{+}^{n_0}$ 与 $\mathbf{b}\in\mathbb{R}_{+}^{n_p}$ 为控制池与扰动池的边缘分布，满足 $\sum_i a_i=\sum_j b_j=1$。带熵正则的 OT 目标写为

$$
\mathbf{P}^{\star}
=
\arg\min_{\mathbf{P} \in \Pi(\mathbf{a}, \mathbf{b})}
\left[
\langle \mathbf{P}, \mathbf{C} \rangle
- \varepsilon H(\mathbf{P})
\right],
$$

其中 $\Pi(\mathbf{a}, \mathbf{b})=\{\mathbf{P}\ge 0 \mid \mathbf{P}\mathbf{1}=\mathbf{a},\mathbf{P}^{\top}\mathbf{1}=\mathbf{b}\}$，$H(\mathbf{P})=-\sum_{i,j}P_{ij}(\log P_{ij}-1)$ 为熵项，$\varepsilon>0$ 为正则系数。实际实现中，TriShift 使用 Sinkhorn 迭代求解近似耦合 $\mathbf{P}^{\star}$，随后按每个扰动细胞对应列从 $\mathbf{P}^{\star}$ 中提取 top-k 控制索引与权重。这里的 top-k 提取是基于耦合矩阵的实现性后处理，而不是 OT 目标本身的一部分。OT 在这里的意义不是寻找唯一正确配对，而是在群体层面找到与当前扰动样本更一致的控制参考；相比随机配对，它显式利用潜空间结构；相比纯硬匹配，它保留了对多模态控制池的软分配能力。

基于 top-k 匹配结果，TriShift 可以构造控制原型

$$
\tilde{\mathbf{z}}^{(0)}=\sum_{j\in\mathcal{M}}w_j\mathbf{z}^{(0)}_j,
$$

其中 $\mathcal{M}$ 表示选中的 top-k 控制参考集合，$w_j$ 表示归一化后的 OT 权重。在此基础上定义潜在位移监督目标

$$
\Delta \mathbf{z}^{\star} = \mathbf{z}^{(p)} - \tilde{\mathbf{z}}^{(0)}.
$$

若采用 soft-OT 权重，则 $\tilde{\mathbf{z}}^{(0)}$ 为多个控制参考的加权平均。这使 OT 不仅用于检索，也直接参与了位移监督目标的构造。

### 4.5 Expression generation, training objectives and inference

Stage 3 将控制状态、扰动语义与 shift 结果映射回表达空间。对于主文默认实例，生成器采用 residual head，即

$$
\hat{\mathbf{x}}^{(p)} = f_{\mathrm{gen}}(\mathbf{g}_{\mathrm{in}}).
$$

$$
\hat{\mathbf{x}}^{(p)} = \mathbf{x}^{(0)} + f_{\mathrm{gen}}(\mathbf{g}_{\mathrm{in}}).
$$

因此，Stage 1 负责学习稳定潜空间，Stage 2 负责学习 perturbation-induced shift，Stage 3 负责将 shift 转回表达变化。

在 Stage 2 和 Stage 3 中，TriShift 同时优化表达恢复与潜在位移恢复。记 $\mathcal{R}$ 为当前 batch 中出现的扰动条件集合，$B_r$ 为属于条件 $r$ 的细胞索引集合，$S_r$ 为条件 $r$ 的有效评估基因集合，$\omega_{r,g}$ 为基因权重，$\gamma$ 为 GEARS 风格 autofocus 指数，$\lambda_x^{\mathrm{dir}}$ 为表达方向损失权重，则表达损失可写为

$$
\mathcal{L}_{\mathrm{expr}}
=
\frac{1}{|\mathcal{R}|}
\sum_{r \in \mathcal{R}}
\frac{1}{|B_r||S_r|}
\sum_{i \in B_r}
\sum_{g \in S_r}
\omega_{r,g}
\left(
|x^{(p)}_{ig} - \hat{x}_{ig}|^{2+\gamma}
+
\lambda_x^{\mathrm{dir}}
\big[
\mathrm{sign}(x^{(p)}_{ig} - x^{(0)}_{ig})
-
\mathrm{sign}(\hat{x}_{ig} - x^{(0)}_{ig})
\big]^2
\right),
$$

在 Adamson 与 Norman 的主配置中，`predict_delta=false`，因此主文默认实例并不直接监督显式 $\Delta \mathbf{z}$ 预测，而是将 $\mathcal{L}_z$ 作为通用训练形式保留给具有显式 delta 头的实现版本。若启用显式 delta 预测，则进一步加入潜在位移监督

$$
\mathcal{L}_{z}
=
\mathrm{MSE}\big(\widehat{\Delta \mathbf{z}}, \Delta \mathbf{z}^{\star}\big),
$$

或其 SmoothL1 / latent GEARS 风格变体。联合目标写为

$$
\mathcal{L} = \mathcal{L}_{\mathrm{expr}} + \lambda_z \mathcal{L}_{z}.
$$

推理时，模型首先根据目标条件构造 `genept` 向量 $\mathbf{c}$；随后在训练控制池中执行 reference-conditioned 检索，获得 top-k 控制原型与可能的 OT 权重；再由 shift 模块生成 $\mathbf{h}$；最后通过生成器输出预测表达。若在测试条件上进行多次控制采样并集成预测，可进一步降低单次参考选择的方差。这样的设计使方法和评估能够围绕同一命题展开，即模型是否真正恢复了相对于控制状态的扰动变化，而非仅拟合系统性偏移。

## 5. Experiments

### 5.1 Datasets, baselines and evaluation protocol

本文主线覆盖五个单扰动数据集：Dixit、Adamson、Norman 单扰动子集、Replogle K562 essential 以及 Replogle RPE1 essential。Norman 组合扰动不作为主任务定义的一部分，而被用作 hard-generalization 压力测试，用于刻画模型在 `single / seen2 / seen1 / seen0` 等不同泛化难度下的行为边界。当前实现采用按条件划分的 train/val/test protocol；Adamson 与 Norman 的 test ratio 为 0.2，验证集划分沿 `split_by_condition` 的默认 `val_ratio=0.1` 执行，并在五个随机 split 上做等权聚合。当前代码路径中，train/val/test 的控制细胞允许重叠，但评估阶段始终从 train control pool 中采样参考；每个测试条件使用 300 次 control ensemble 采样。[文献待补：Scouter split protocol] 对比方法包括图结构与知识驱动方法、语言模型嵌入驱动方法以及若干统一评估封装下的基线模型。根据当前仓库与文档安排，主比较对象包括 TriShift nearest、biolord、GEARS、GenePert 以及在共享设置下可纳入的 scGPT。指标方面，本文同时关注幅值恢复、方向一致性、分布拟合和异质性重建四类能力。常规指标包括 DE 基因上的 Pearson、nMSE 和决定系数；围绕 perturbation-specific signal 的分析则重点使用 Systema 风格 Pearson 指标；分布层面的比较使用 Wasserstein 指标；对细胞间异质性的重建则参考 scPRAM 兼容重采样指标。

### 5.2 Main-text figure organization

主文图按照 `Fig1 -> Fig2 -> Fig3 -> Fig4 -> Fig5` 的顺序出现，并分别对应不同层级的论文主张。`Fig1_MethodOverview.ipynb` 用于给出任务重构、方法框图与数据版图，从而明确 TriShift 的问题设定与主线数据集边界。`Fig2_MultiDatasetBenchmark.ipynb` 用于展示五个单扰动数据集上的统一 benchmark，总结跨数据集的一致性增益，并通过代表性 case 补充 shared-baseline 与 coverage 证据。`Fig3_ReferenceConditioning.ipynb` 专门检验 reference-conditioned retrieval 是否构成 TriShift 的关键增益来源，而不仅是普通的网络容量提升。`Fig4_NormanGeneralization.ipynb` 将 Norman subgroup 作为 stress test，用于刻画模型在更困难泛化场景中的边界。`Fig5_BiologyAndAblation.ipynb` 则用于展示 pathway 或 program 层面的 biological recovery，其中主文默认以 Norman pathway summary 为核心。由于本文当前仍是底稿阶段，主文图顺序已经固定，但个别面板是否进入最终版本仍取决于尚未完成的实验结果。

### 5.3 Supplementary analyses

补充材料按 `FigS1 / FigS2 / FigS3 / FigS4 / FigS7` 的顺序组织。FigS1 提供完整 benchmark 表；FigS2 提供额外单扰动与组合扰动案例，以降低 cherry-pick 风险；FigS3 进行 condition centroid analysis；FigS4 汇报 stratified robustness；FigS7 则用于说明 Stage 1 学到了具有生物学意义的细胞表示。文末 Supplement 还将系统解释本文使用的评估指标，并说明这些补充分析与主文图之间的对应关系。

## 6. Results

### 6.1 Multi-dataset benchmark

TriShift 的第一项核心主张是：其收益应具有跨数据集一致性，而不是仅在个别 benchmark 上成立。基于这一主张，我们首先在五个单扰动数据集上评估 TriShift 的主任务表现，对应 `Fig2_MultiDatasetBenchmark.ipynb`。Fig2a 用于给出统一 benchmark 总览，Fig2b 用于检验这种增益是否仍然成立于 perturbation-specific 指标，Fig2c 则把多数据集结果收束为 dataset-level summary。Fig2d 与 Fig2e 分别补充 shared-baseline 条件和超出 GEARS coverage 的特殊条件。具体数值、排名与显著性表述仍待实验完成后补充。

### 6.2 Evidence for reference-conditioned modeling

TriShift 的第二项核心主张是：性能提升来自 reference-conditioned matching，而不是普通的网络容量增加。为检验这一点，我们在 `Fig3_ReferenceConditioning.ipynb` 中组织了专门的机制性分析。Fig3a 比较 nearest retrieval 与 random retrieval，以直接检验 inference-time retrieval 的有效性；Fig3b 将 TriShift 与若干 reference baseline 进行对照；Fig3c 讨论 retrieval gain 与条件难度的关系；Fig3d 通过 reference-sensitive case 给出更直观的案例说明。此处预留实验结果与图表说明。

### 6.3 Hard generalization on Norman

TriShift 的第三项核心主张是：在更困难的泛化场景中，参考条件化建模仍应提供可解释的收益边界。为此，我们将 Norman subgroup 作为 stress test，对应 `Fig4_NormanGeneralization.ipynb`。Fig4a 说明 subgroup 任务定义；Fig4b 比较 `seen2 / seen1 / seen0` 等不同泛化难度下的 benchmark 表现；Fig4d 给出 unseen combo 的代表性案例。Fig4c 是否进入主文，将取决于最困难 regime 的结果是否足够稳定。该部分结果待补充。

### 6.4 Biological recovery and supporting analyses

TriShift 的第四项核心主张是：模型改进应延伸到更接近生物学解释的层面，而不仅停留在均值相关性上。基于这一主张，我们在 `Fig5_BiologyAndAblation.ipynb` 中考察 pathway 或 program 层面的 biological recovery。Fig5a 固定展示 Norman pathway 或 program summary，Fig5b 给出代表性 pathway case。Adamson pathway 分析更适合作为补充材料 reserve。与之配套的 supporting evidence 将在补充图中给出，包括完整 benchmark 表、额外案例、condition centroid analysis、stratified robustness 以及 Stage 1 latent clustering。具体 pathway 统计、代表性案例与定量比较仍待补充。

## 7. Discussion

TriShift 所强调的并不是某一种特定网络算子，而是一种更适合单细胞扰动预测的建模视角：将扰动响应理解为参考条件化状态转移，而不是仅把扰动条件映射到表达终点。这一视角使控制参考、潜在位移与表达生成可以被统一地组织起来，也使评估能够更自然地围绕 perturbation-specific signal 展开。

Systema 所指出的系统性偏移问题，为本文提供了重要的方法学背景。若模型只是在整体层面逼近真实表达，而没有正确恢复相对于控制状态的扰动变化，那么常规相关性指标可能会高估模型能力。TriShift 的设计正是试图缩小这一偏差：一方面通过 reference-conditioned retrieval 改善控制参考，另一方面通过 `systema_Pearson` 等指标显式检验模型是否真正超越系统性偏移。

本文方法也存在需要在定稿中进一步说明的限制。首先，参考匹配质量依赖于 Stage 1 潜空间的稳定性；若潜空间学习不足，后续 OT 匹配和 delta 监督都会受影响。其次，GenePT 式语义嵌入虽然有助于外推，但其质量本身依赖预训练语料与嵌入模型。再次，Norman subgroup 作为 stress test 具有重要价值，但其任务属性与单扰动主线并不完全相同，因此相关结果需要谨慎解释。最后，pathway 级分析对通路集合与富集方法较为敏感，正式版本需要补充更完整的方法细节。

## 8. Conclusion

本文提出 TriShift，将单细胞扰动预测重新表述为参考条件化状态转移问题。不同于直接的 condition-to-expression 映射，TriShift 通过 `state`、`genept` 和 `latent_mu` 的分工式表示，将控制匹配、潜在位移学习与表达生成组织为统一链路；同时引入 OT 风格匹配以在群体层面构造更合适的控制参考。围绕主文图顺序，本文进一步设计了跨数据集 benchmark、reference-conditioned 证据、Norman hard-generalization 和 pathway biological recovery 四条结果主线。当前稿件已给出较完整的方法与实验框架，待相关结果整理完成后，将进一步补充定量比较、统计检验与文献引用。

## 9. Data Availability

本文使用的主要数据来自公开单细胞扰动数据集，包括 Dixit、Adamson、Norman、Replogle K562 essential 和 Replogle RPE1 essential。当前仓库已经整理相应的预处理文件、配置与训练脚本；正式投稿版本中将进一步补充数据访问链接、版本信息、预处理说明以及与图表对应的数据导出路径。

## 10. Code Availability

TriShift 的核心实现、训练与评估脚本、基线封装以及论文图 notebook 已在当前研究仓库中整理。正式版本的代码可用性声明将根据投稿阶段的匿名要求进行调整，并在公开版本中提供复现实验所需的配置文件、运行入口、随机划分设置以及结果导出脚本。

## Supplement

### S1. Evaluation metrics

本文的评估指标分为四类：幅值恢复、方向一致性、扰动特异信号恢复，以及分布与异质性重建。设置这一补充说明的原因在于，主文中不同图使用的指标具有不同侧重点；若不显式解释，读者很容易把所有相关性指标视为等价。这里采用“论文术语 + 仓库字段映射”的双层写法，以避免论文名词与代码输出字段不一致。

设真实扰动均值、预测扰动均值和控制均值分别为 $\boldsymbol{\mu}_t$、$\boldsymbol{\mu}_p$ 与 $\boldsymbol{\mu}_0$，DE 基因集合记为 $D$。本文在 DE 基因上计算扰动增量

$$
a_g = \mu_t[g] - \mu_0[g], \qquad b_g = \mu_p[g] - \mu_0[g], \qquad g \in D.
$$

DE Pearson 定义为

$$
\mathrm{Pearson}_{\mathrm{DE}} = \rho(\mathbf{a}, \mathbf{b}),
$$

用于衡量预测是否恢复了 DE 基因上的扰动方向与相对排序。与之对应，nMSE 定义为

$$
\mathrm{nMSE}
=
\frac{\frac{1}{|D|}\sum_{g \in D}(\mu_t[g]-\mu_p[g])^2}
{\frac{1}{|D|}\sum_{g \in D}(\mu_t[g]-\mu_0[g])^2},
$$

用于衡量模型相对于“直接输出控制均值”这一弱基线的幅值恢复能力。前者更关注方向一致性，后者更关注绝对幅值校准。除均值层面指标外，本文还使用 Wasserstein 距离评估预测分布与真实分布之间的差异，以反映模型是否保留了群体层面的表达结构。与此同时，我们保留 scPRAM 兼容的重采样指标，用于评估在随机子采样下，真实细胞与预测细胞在均值、方差和 DE 子空间上的一致性。这些指标更接近“分布恢复”而不是“单个均值恢复”，因此适合与 OT 匹配和群体对齐的建模视角配合使用。

### S2. Why `systema_Pearson`

为降低系统性偏移导致的偏乐观估计，本文引入论文层面的记号 `systema_Pearson`。需要强调的是，这一术语在当前仓库中并不是单一列名，而是对一类 Systema 风格 Pearson 指标的统称。与普通 DE Pearson 的关键区别在于，普通 Pearson 比较的是相对于条件特异控制均值 $\boldsymbol{\mu}_0$ 的增量，即 $x-\!x_{\mathrm{ctrl}}$；而 `systema_Pearson` 比较的是相对于参考向量 $\mathbf{r}$ 的增量。当前实现中，$\mathbf{r}$ 不是条件特异控制均值，而是 Systema 风格的 perturbation centroid reference，即对所有非控制扰动条件的质心向量取平均得到的全局参考向量：

$$
\mathbf{r}
=
\frac{1}{|\mathcal{C}_{\mathrm{pert}}|}
\sum_{c' \in \mathcal{C}_{\mathrm{pert}}}
\boldsymbol{\mu}_{c'},
\qquad
\boldsymbol{\mu}_{c'}=\frac{1}{N_{c'}}\sum_{i=1}^{N_{c'}}\mathbf{x}^{(c')}_i .
$$

相应地，普通控制差分和 Systema 差分分别为

$$
\delta^{\mathrm{ctrl}} = \boldsymbol{\mu}_{t} - \boldsymbol{\mu}_0,
\qquad
\delta^{\mathrm{sys}} = \boldsymbol{\mu}_{t} - \mathbf{r}.
$$

设参考向量为 $\mathbf{r}$，则真实和预测相对于参考的增量分别为

$$
\Delta_t[g] = \mu_t[g] - r[g], \qquad \Delta_p[g] = \mu_p[g] - r[g].
$$

论文层面的 `systema_Pearson` 定义为

$$
\mathrm{systema\_Pearson} = \rho(\Delta_t, \Delta_p).
$$

当该指标在全基因空间计算时，它衡量模型是否恢复了参考差分后的整体扰动方向；当在 top-DE 子集上计算时，它更强调最显著扰动信号的恢复。与普通的 $x-\!x_{\mathrm{ctrl}}$ 口径相比，Systema 口径试图扣除的是“相对于公共参考的偏移”，而不是“相对于该条件控制均值的偏移”。因此，两者回答的问题并不相同：前者更接近条件特异效应恢复，后者更强调是否超越了全局系统性偏移。当前仓库中，与论文术语 `systema_Pearson` 对应的主要输出字段包括 `systema_corr_20de_allpert`（主文 signal metric，top-20 DE 上的 Pearson）、`systema_corr_all_allpert`（全基因版本，主要在分析脚本中出现）以及 `systema_corr_deg_r2`（同一参考口径下的回归 $R^2$ 伴随指标）。对本文而言，`systema_Pearson` 并不是一个额外装饰指标，而是用于回答“模型是否真的超越了 systematic variation”的核心证据。

### S3. Supplementary figures

补充图按 `FigS1 / FigS2 / FigS3 / FigS4 / FigS7` 的顺序组织。FigS1 用于给出主文未展开的完整 benchmark 表；FigS2 用于提供额外单扰动与组合扰动案例，降低 cherry-pick 风险；FigS3 用于展示 condition-level structure recovery；FigS4 用于提供 stratified robustness 证据；FigS7 用于说明 Stage 1 潜空间的生物学可解释性。与主文图的对应关系大致如下：Fig2a 主要使用 Pearson、nMSE 和决定系数来展示跨数据集主 benchmark；Fig2b 和 Fig3a/3b 重点使用 `systema_Pearson` 来检验 perturbation-specific recovery；Fig3c 可结合距离分层指标分析 retrieval gain 与难度的关系；Fig5 则优先使用 pathway 或 program-level 指标，如 NES Pearson、NES Spearman 与 pathway hit@k。这样的安排使每张图的指标都服务于其对应主张，而不是用单一指标覆盖全文。
