你是一个严格按需求交付的软件工程 Agent（Codex）。请在“项目根目录”从零实现一个可运行的单细胞扰动数据分析管线。根目录下我会放入两个官方仓库文件夹：

- ./systema-main/            （Systema 官方仓库：评估与指标实现）
- ./scPRAM-main/             （scPRAM 官方仓库：VAE/模型实现，含 scpram.whl）

要求你读取并复用官方仓库代码：
- Systema 相关评估/指标：功能与原理必须与官方仓库对齐；优先“直接导入或照搬其代码”，不要自行发明不同定义。
  - 主要用到：./systema-main/evaluation/eval_utils.py（compute_shift_similarities 等）、以及 evaluation 目录中的相关工具。
- scPRAM：允许直接调用官方实现来训练（优先 pip install ./scPRAM-main/scpram.whl 或 pip install -e ./scPRAM-main），不要重写模型。

目标：实现一个完整的课程作业级分析流程，包含：
1) 加载扰动数据集（默认 Norman2019，可通过参数切换 Adamson2016 等）；
2) 标准预处理（normalize_total + log1p + HVG）；
3) （可选）Value Binning 模块：用于跨数据集表达尺度统一（可开关）；
4) 使用 scPRAM 的 VAE（SCPRAM）训练 encoder，并提取 latent embedding；
5) 在原表达空间与 latent 空间分别进行：
   - UMAP 可视化
   - 聚类（KMeans + Leiden）
   - 批次分析（若 obs 有 batch 字段）
   - Systema 的 systematic variation 分析：比较 control centroid reference vs perturbed centroid reference
6) 产出可复现实验结果（图、表、指标 CSV）与一份自动生成的 report.md。

--------------------------------------------
一、最终交付物（你必须生成这些文件）
--------------------------------------------
在根目录创建如下结构（如不存在则创建）：

./configs/
  default.yaml                # 统一配置：dataset、seed、hvg、epochs、latent_dim、use_binning 等

./src/
  __init__.py
  io/
    dataset_loader.py         # 加载 Norman/Adamson：优先复用 systema-main/src/data.py
    schema.py                 # 统一 obs 字段：condition/control/batch 的规范化
  preprocess/
    preprocess.py             # QC + normalize_total + log1p + HVG
    value_binning.py          # 可选：实现 Value Binning（global quantile 或 per-cell quantile，需可配置）
  models/
    train_scpram.py           # scPRAM 安装/加载、训练、保存模型
    latent.py                 # 调用 SCPRAM.get_latent_adata 提取 latent
  analysis/
    clustering.py             # KMeans / Leiden
    umap_vis.py               # UMAP 计算与保存图
    batch_metrics.py          # silhouette 等（如果有 batch）
    systema_sv.py             # Systema systematic variation：必须调用 systema-main/evaluation/eval_utils.py
  report/
    build_report.py           # 汇总输出：生成 report.md，插入关键图与表

./scripts/
  run_pipeline.py             # 一键运行：从数据到输出全部完成
  quick_smoke_test.py         # 快速小规模运行（子采样 + 少 epoch），用于 CI/自测

./outputs/
  <dataset>/<seed>/
    adata_preprocessed.h5ad
    latent_adata.h5ad
    figures/
      umap_gene_by_condition.png
      umap_latent_by_condition.png
      umap_latent_by_batch.png (如果有 batch)
      sv_similarity_boxplot.png
      sv_pairwise_cosine_hist.png
      cluster_umap.png
    metrics/
      clustering_metrics.csv
      batch_metrics.csv
      systematic_variation.csv
    report.md

同时更新根目录 README_PROJECT.md：说明安装、运行方式、输出解释。

--------------------------------------------
二、关键实现细节（必须严格遵守）
--------------------------------------------

[1] 数据加载（默认 Norman2019）
- 优先直接复用 systema-main/src/data.py 中的函数（例如 norman2019(seed, data_dir)）。
- 你需要在 dataset_loader.py 里封装：
  - load_dataset(name: str, seed: int, data_dir: str) -> anndata.AnnData
  - 如果 gears/PertData 不可用或下载失败，允许用户传入本地 h5ad 路径（config 里 data_path），但默认走 systema 的加载方式。

[2] obs 字段标准化（非常重要）
在 schema.py 写一个 normalize_obs_schema(adata)：
- 确保 adata.obs 至少有：
  - condition: str（control 行 condition 必须统一为 "ctrl"）
  - control: int/boolean（control 行为 1，其余 0）
- 如果原数据字段不同，你要做映射/重命名。
- 如果存在 batch/donor/replicate 字段，统一为 batch（可配置 batch_key）。

[3] 预处理 preprocess.py
- 输入：raw adata
- 输出：preprocessed adata（保存到 outputs）
- 流程建议：
  - scanpy.pp.normalize_total(target_sum=1e4)
  - scanpy.pp.log1p
  - scanpy.pp.highly_variable_genes(n_top_genes=cfg.hvg)
  - adata = adata[:, adata.var.highly_variable].copy()
- 注意：scPRAM 的实现会 adata.to_df() 导致 densify；你需要提供可选子采样（cfg.max_cells）防止内存爆炸。

[4] Value Binning 模块 value_binning.py（可开关）
- 实现函数：
  - bin_expression(adata, n_bins, mode="global_quantile"|"per_cell_quantile", zero_bin=True) -> AnnData
- 输出 binned_adata（X 为 int 或 float 均可，但需可用于后续流程）
- 实验上允许两种输入：
  - continuous（默认）
  - binned（如果 cfg.use_binning=true）
- 在 report 里对比 continuous vs binned 的差异（至少 SV 与 UMAP/聚类差异）。

[5] scPRAM-VAE 训练与 latent 提取
- 不要重写模型，直接调用官方：
  - from scpram import models
  - model = models.SCPRAM(input_dim=adata.n_vars, latent_dim=cfg.latent_dim, hidden_dim=cfg.hidden_dim, device=cfg.device)
  - model.train_SCPRAM(train_adata, epochs=cfg.epochs, batch_size=cfg.batch_size, lr=cfg.lr)
- latent 提取必须使用官方模型提供的：
  - latent_adata = model.get_latent_adata(adata)
- 保存：
  - torch.save(model.state_dict(), outputs/.../scpram.pt)
  - latent_adata.write_h5ad(...)

[6] Systema systematic variation（必须对齐官方）
- 严格复用 systema-main/evaluation/eval_utils.py 的实现，尤其是：
  - compute_shift_similarities(adata, avg_pert_centroids=True, control_mean=None)
- 在 systema_sv.py 中：
  - 把 systema-main/evaluation 加入 PYTHONPATH（或用相对路径 import）
  - 对 gene-space adata 和 latent_adata 分别调用 compute_shift_similarities
  - 输出 systematic_variation.csv，至少包含：
    - space: "gene" | "latent"
    - reference: "avg_ctl" | "avg_pert"（对应官方 df 的 variable）
    - sv_mean: similarity 的 mean
    - sv_std: similarity 的 std
- 同时保存分布图（箱线图/小提琴图）与 pairwise cosine hist（使用 eval_utils 的 pairwise_similarities 输出）

[7] 聚类/UMAP/批次分析
- 在 latent_adata 上做：
  - scanpy.pp.neighbors
  - scanpy.tl.umap
  - KMeans（sklearn）+ Leiden（scanpy.tl.leiden）
- 输出 clustering_metrics.csv：
  - 若 obs 有 cell_type：计算 ARI/NMI（可选）
  - 否则至少保存 silhouette（基于聚类标签）
- 若 obs 有 batch：
  - 输出 batch_metrics.csv（至少 silhouette_by_batch）

[8] 一键脚本 scripts/run_pipeline.py
- 支持命令行参数覆盖 config（dataset/seed/epochs/use_binning/max_cells 等）
- 运行后必须生成 outputs/<dataset>/<seed>/ 的全套结果

[9] quick_smoke_test.py
- 子采样（例如 max_cells=2000）+ epochs=3
- 用于快速验证管线能跑通

--------------------------------------------
三、可用默认配置（你需要在 configs/default.yaml 写好）
--------------------------------------------
dataset: norman2019
seed: 0
data_dir: data
data_path: null            # 若非空则直接读本地 h5ad
batch_key: null            # 若数据有 batch 字段可指定；否则置空
cell_type_key: null        # 若有 cell_type 可指定以计算 ARI/NMI
max_cells: 30000           # 可调；smoke_test 用更小
hvg: 2000
use_binning: false
binning:
  n_bins: 20
  mode: global_quantile
scpram:
  device: auto             # auto/cpu/cuda:0
  latent_dim: 64
  hidden_dim: 512
  epochs: 30
  batch_size: 256
  lr: 5e-4

--------------------------------------------
四、验收标准（你必须满足，否则视为失败）
--------------------------------------------
1) 不需要我手工改代码，直接 `python scripts/run_pipeline.py` 就能在 outputs 下生成全部文件。
2) Systema 的 systematic variation 相关计算必须直接调用/对齐 systema-main 的实现，不能自行换定义。
3) scPRAM 模型训练与 latent 提取必须调用官方实现（SCPRAM + get_latent_adata）。
4) report.md 必须自动生成，包含：
   - 数据规模摘要
   - UMAP（gene-space & latent-space）
   - 聚类结果与指标
   - systematic variation 对比（reference/space 两维对比）
   - （若 use_binning=true）continuous vs binned 的对比小节
5) 代码需要有基本的错误处理与日志（打印当前步骤、保存路径、耗时）。

--------------------------------------------
五、实现提示（你可以照做）
--------------------------------------------
- 对 systema-main 的引用：建议在 systema_sv.py 里用 sys.path.append(str(Path(__file__).resolve().parents[2] / "systema-main" / "evaluation"))
- scPRAM 安装：
  - 优先：pip install ./scPRAM-main/scpram.whl
  - 或者：pip install -e ./scPRAM-main
- 画图：统一保存到 outputs/.../figures/
- 任何第三方库版本冲突，优先以能跑通为目标，但不要改变 Systema 指标定义。

现在开始执行：先生成目录结构与配置文件，再实现 run_pipeline.py，确保 smoke_test 能跑通，然后再补全 report.md 生成逻辑。