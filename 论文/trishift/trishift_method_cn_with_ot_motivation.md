# TriShift：面向单细胞基因扰动预测的参考条件化状态转移建模

## 4. Method

### 4.1 问题设定与 TriShift 总体框架

设基因表达空间为 $\mathcal{X}\subset\mathbb{R}^{G}$，其中 $G$ 为基因维度。对于每个扰动条件 $c\in\mathcal{C}$，我们观测控制细胞集合 $\{x_j^{(0)}\}_{j=1}^{N_0}$ 与扰动细胞集合 $\{x_i^{(p)}\}_{i=1}^{N_p}$。TriShift 的目标不是直接学习从扰动条件到扰动后表达终点的静态映射，而是学习一个**参考条件化（reference-conditioned）的状态转移过程**：在给定参考控制状态的条件下，预测扰动如何推动细胞状态迁移，并进一步生成扰动后的基因表达。

这一任务表述对应默认模型的基本假设：单细胞扰动响应首先取决于“从哪个控制状态出发”，其次才取决于“施加的扰动是什么”。因此，TriShift 将单细胞基因扰动预测组织为一条顺序明确的建模链路：首先学习可比较的控制状态表示；随后构造与当前扰动更匹配的控制参考；再结合 GenePT 扰动语义学习参考条件化的状态位移；最后生成扰动后的表达。形式上，TriShift 预测
$$
\hat{x}^{(p)} = F\bigl(z_{\mathrm{state}}^{(0)}, c, h\bigr),
$$
其中 $z_{\mathrm{state}}^{(0)}$ 表示控制细胞状态表示，$c$ 表示扰动条件的 GenePT 语义向量，$h$ 表示由参考状态与扰动语义共同诱导的紧凑位移表征，$F(\cdot)$ 表示表达生成器。与直接的 condition-to-expression 回归不同，TriShift 将“参考匹配—位移建模—表达生成”组织为同一条主链路，使参考控制状态成为方法定义的一部分。

按照默认配置，TriShift 的模型实例具有如下特征。Stage 1 使用去噪变分自编码器学习潜空间表示；Stage 2 使用 **Transformer 化的 shift 模块**，并以控制细胞潜均值作为输入，而不是采用显式 $\Delta z$ 回归头；Stage 3 使用 **full 输入模式**，同时接收控制状态、GenePT 条件向量与 shift 表征，并采用 **残差式表达生成头** 输出扰动后表达。推理阶段默认同时报告随机控制参考与 **GenePT 引导的控制池路由** 两条路径，后者先进行语义最近邻检索，再复用相应条件的 OT 控制池。

### 4.2 学习可比较的控制状态表示

TriShift 首先使用去噪变分自编码器（denoising variational autoencoder, VAE）学习单细胞表达的低维潜空间表示。给定输入表达向量 $x$，模型首先注入高斯噪声：
$$
\tilde{x} = x + \epsilon,\qquad \epsilon\sim\mathcal{N}(0,\sigma_n^2 I).
$$
随后由编码器参数化近似后验分布：
$$
q_{\phi}(z\mid \tilde{x})=\mathcal{N}\bigl(\mu_{\phi}(\tilde{x}),\mathrm{diag}(\exp(\log \sigma_\phi^2(\tilde{x})))\bigr).
$$
通过重参数化采样得到潜变量 $z$，并由解码器重构输入表达。Stage 1 的训练目标为
$$
\mathcal{L}_{\mathrm{vae}}
=
\frac{1}{B}\sum_{b=1}^{B}
\left[
\frac{1}{2}\lVert x_b-\hat{x}_b\rVert_2^2
+
\frac{1}{2}\beta\lambda_{\mathrm{kl}}
\mathrm{KL}\bigl(q_{\phi}(z\mid\tilde{x}_b)\Vert \mathcal{N}(0,I)\bigr)
\right].
$$

默认模型中，Stage 1 产出两类后续使用的状态表示。第一类是潜均值
$$
z_{\mu}=\mu_{\phi}(x)\in\mathbb{R}^{d_z},
$$
它主要用于参考匹配、OT 控制池构造以及 shift 模块输入。第二类是控制状态表示
$$
z_{\mathrm{state}} = f_{\mathrm{enc}}(x^{(0)})\in\mathbb{R}^{d_s},
$$
它由 Stage 3 的压缩器从控制表达中提取，用于表达生成阶段保留细胞的基础状态背景。默认配置下，$z_\mu$ 负责“潜空间定位与匹配”，$z_{\mathrm{state}}$ 负责“表达生成时的状态保真”。

### 4.3 使用 GenePT 表示扰动条件

TriShift 使用基于大语言模型（large language model, LLM）的 GenePT 基因语义嵌入表示扰动条件。与将扰动编码为离散标签或独热向量不同，GenePT 将基因功能、事实描述和注释文本编码为连续向量，从而为未见基因和未见条件提供可迁移的语义结构。

对于扰动条件 $c$，记其包含的基因集合为 $\mathcal{G}(c)$，每个基因 $g$ 对应一个 GenePT 向量 $e_g\in\mathbb{R}^{d_c}$。默认配置下，TriShift 使用 **sum pooling** 聚合多个基因的语义表示：
$$
c=\sum_{g\in\mathcal{G}(c)} e_g.
$$
当前默认模型不再对条件向量额外做 $L_2$ 归一化，而是将其直接送入后续的 shift 模块和推理期的语义最近邻检索。这样，GenePT 在 TriShift 中同时承担两项职责：提供扰动语义，并在推理阶段参与控制池路由。

### 4.4 构造参考控制状态

TriShift 的核心不是无条件地从训练控制池中随机抽取参考，而是为每个扰动样本构造更匹配的控制参考。默认模型在训练阶段与推理阶段分别采用两种互相衔接的参考构造机制：训练阶段以潜空间匹配和 OT 参考池为主；推理阶段进一步引入 GenePT 引导的控制池路由。

在单细胞扰动数据中，控制细胞与扰动细胞通常是以群体形式分别测量的，而非同一细胞在扰动前后的配对观测。由于单细胞测序具有破坏性，我们无法直接获得真实的 cell-to-cell correspondence，因此参考构造更自然地应被表述为分布层面的匹配问题，而不是样本层面的硬配对。TriShift 并不假设扰动后的细胞能够与扰动前的某个细胞形成真实且唯一的严格配对；相反，它只使用一个更弱的假设：扰动后的细胞状态虽然发生变化，但通常仍与其扰动前基础状态保持局部连续关系，因此在一个可比较的潜空间中，控制细胞分布与扰动细胞分布之间存在可恢复的软对应关系。基于这一动机，TriShift 使用最优传输（optimal transport, OT）来学习控制分布与扰动分布之间的耦合关系，并据此构造 matched control references，而不是直接恢复并不存在的真实单细胞配对。

#### 4.4.1 基于潜空间与最优传输的训练期参考构造

给定控制潜向量集合 $\{z_i^{(0)}\}_{i=1}^{n_0}$ 与扰动潜向量集合 $\{z_j^{(p)}\}_{j=1}^{n_p}$，TriShift 在潜空间中定义代价矩阵
$$
C_{ij}=\lVert z_i^{(0)}-z_j^{(p)}\rVert_2.
$$
默认训练配置采用 **OT matching**，即在边缘分布 $a\in\mathbb{R}_+^{n_0}$ 与 $b\in\mathbb{R}_+^{n_p}$ 下求解带熵正则项的最优传输问题：
$$
P^{\star}
=
\arg\min_{P\in\Pi(a,b)}
\left[
\langle P,C\rangle-\varepsilon H(P)
\right],
$$
其中 $\Pi(a,b)=\{P\ge 0\mid P\mathbf{1}=a,\;P^\top\mathbf{1}=b\}$。TriShift 使用 OT 耦合矩阵为每个扰动样本构造 top-$k$ 控制参考池，并从中得到与该扰动更一致的控制原型
$$
\tilde{z}^{(0)}=\sum_{j\in\mathcal{M}} w_j z_j^{(0)}.
$$
对于本文主线实验，Norman 与 Adamson 的默认配置都采用 OT 作为训练期参考构造方式，只是在 top-$k$ 数量与是否按条件分别求 OT 上存在数据集级别差异。

需要强调的是，这里的 OT 在 TriShift 中承担的是**参考构造**而不是**最终预测**的角色。它的作用不是直接把控制细胞映射为扰动细胞，而是在潜空间中为后续的条件化位移建模提供一个更合理的参考起点。换言之，TriShift 使用 OT 回答的是“从哪个参考控制状态出发更合适”，而不是“OT 本身就是预测器”。

#### 4.4.2 GenePT 引导的推理期控制池路由

默认推理配置不仅保留随机控制参考作为对照，还引入一条 **GenePT 引导的控制池路由策略**。其核心思想是：面对未见扰动时，模型不应直接在整个训练控制池中无条件采样参考，而应先在训练扰动条件中找到与目标扰动在 GenePT 语义空间中更接近的条件，再复用这些已知条件对应的 OT 控制池，从而构造更合理的参考控制分布。

默认配置下，该路由过程使用 **$L_2$ 距离** 比较 GenePT 语义相似性，并采用 **per-gene nearest condition** 模式。对于组合扰动，TriShift 不是先将整个条件向量聚合后只做一次最近邻匹配，而是对每个组成基因分别寻找语义最近的训练条件，再将这些最近条件对应的 OT 控制池拼接起来，形成目标扰动的候选控制池。默认候选集模式设为 **norman_train_single_only**，即优先在 Norman 当前 split 的单扰动训练条件中寻找语义最近邻；若当前数据集不适用该约束或候选集为空，则再回退到更一般的训练扰动候选集合。

这种设计使 TriShift 在推理阶段形成一个两级参考构造过程：首先利用 GenePT 进行**条件层面的语义参考检索**，再利用训练期已构造的 OT 控制池进行**细胞层面的参考复用**。

### 4.5 建模参考条件化的状态位移

在获得参考控制状态与扰动语义之后，TriShift 进一步学习扰动如何推动参考状态发生迁移。默认模型中，Stage 2 不直接预测显式的潜在位移 $\Delta z$，而是输出一个**低维紧凑 shift 表征**：
$$
h=f_{\mathrm{shift}}(z_{\mu}^{(0)}, c),\qquad h\in\mathbb{R}^{8}.
$$
这里 $z_\mu^{(0)}$ 为控制侧潜均值，$c$ 为上节定义的 GenePT 条件向量。默认配置下，shift 模块采用 **Transformer block**，而不是纯 MLP。模型关闭 cross-attention，采用单层 Transformer block，对控制状态与扰动语义进行条件交互，并使用 **concat readout** 读出最终的 shift 表征。

这一写法与当前默认配置严格一致：模型开启 `predict_shift`，但关闭 `predict_delta`，因此 Stage 2 学到的是供生成器使用的条件化位移表征，而不是显式的 $\widehat{\Delta z}$。默认模型不再把“准确回归一个潜空间增量向量”作为 Stage 2 的直接目标，而是将 shift 表征视为连接控制状态与表达生成的中间桥梁。

### 4.6 由控制状态与 shift 表征生成扰动后表达

在完成状态位移建模之后，TriShift 通过表达生成器输出扰动后的基因表达。默认 Stage 3 使用 **full 输入模式**，即同时接收 GenePT 条件向量、控制状态表示与 shift 表征：
$$
g_{\mathrm{in}} = [c;\, z_{\mathrm{state}};\, h].
$$
生成器输出为
$$
\hat{x}^{(p)} = f_{\mathrm{gen}}(g_{\mathrm{in}}).
$$
在默认配置下，Stage 3 开启 **残差式表达生成头**，因此最终预测写为
$$
\hat{x}^{(p)} = x^{(0)} + f_{\mathrm{gen}}(g_{\mathrm{in}}).
$$
这意味着模型显式学习相对于控制表达的变化量，而不是从零生成一个新的表达谱。这样的设计与 TriShift 的任务定义保持一致：真正需要建模的是“扰动相对于参考控制状态带来了什么变化”。

### 4.7 训练目标与推理流程

默认配置下，TriShift 的主训练目标聚焦于**表达恢复**，而不是显式的潜在位移监督。原因在于默认 Stage 2 关闭了 `predict_delta`，因此不再单独回归 $\widehat{\Delta z}$。换言之，shift 表征 $h$ 通过表达恢复损失间接学习，而不是通过单独的 latent displacement loss 显式监督。

对于表达预测，TriShift 在当前批次中对各扰动条件的有效评估基因集合 $S_r$ 计算加权表达损失。记 $\omega_{r,g}$ 为基因权重，$\gamma$ 为 autofocus 指数，$\lambda_{\mathrm{dir}}^x$ 为方向损失权重，则默认训练目标写为
$$
\mathcal{L}_{\mathrm{expr}}
=
\frac{1}{|R|}
\sum_{r\in R}
\frac{1}{|B_r||S_r|}
\sum_{i\in B_r}
\sum_{g\in S_r}
\omega_{r,g}
\left(
|x_{ig}^{(p)}-\hat{x}_{ig}|^{2+\gamma}
+
\lambda_{\mathrm{dir}}^x
\bigl[
\mathrm{sign}(x_{ig}^{(p)}-x_{ig}^{(0)})
-
\mathrm{sign}(\hat{x}_{ig}-x_{ig}^{(0)})
\bigr]^2
\right).
$$
默认训练模式为 **joint**。因此，在完成 Stage 1 的潜空间学习后，Stage 2 与 Stage 3 作为一条连续链路联合优化：训练期使用 OT 构造参考控制池，Stage 2 生成紧凑 shift 表征，Stage 3 基于 full 输入与残差头恢复扰动后表达。推理阶段则分为两条参考路径：一条是随机训练控制参考，用作标准对照；另一条是本文强调的 **GenePT 引导推理期控制池路由**，即先在训练扰动条件中进行语义最近邻检索，再复用其 OT 控制池。对于后者，默认设置还对组合扰动采用逐基因最近条件匹配，从而在未见扰动场景下提供更细粒度的参考构造机制。

综上，按默认配置实例化的 TriShift 可以被理解为一个顺序明确的参考条件化生成框架：先学习控制状态表示，再以 OT 构造训练期参考，以 GenePT 路由推理期参考，随后用 Transformer 化 shift 模块学习状态位移表征，最后通过 full + residual 的生成器输出扰动后的表达预测。
