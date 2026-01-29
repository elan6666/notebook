# GenePert 论文复现笔记

本笔记聚焦代码结构与函数调用关系，重点解释 `GenePertExperiment.py` 的实现逻辑。

---

## 仓库文件概览（与复现实验相关）

- `GenePertExperiment.py`：**核心实验逻辑**（数据加载、特征构建、训练、评估、交叉验证）。
- `utils.py`：**实验对比与可视化**（多 embedding 对比、绘图、结果汇总）。
- `genepert-*-demo.ipynb`：**示例运行入口**（Adamson / Norman / K562）。
- `run_genepert_biolord.py`：**脚本化运行入口**（批量跑 Adamson/Norman）。
- `README.md` / `README_zh.md`：项目说明。

---

## 重点：`GenePertExperiment.py`

### 1) 主要类与职责

#### `GenePertExperiment`
**作用**：把“基因 embedding + 扰动表达数据”转为回归问题，训练并评估模型。

核心字段：
- `self.embeddings`：基因嵌入字典 `{gene_name: embedding_vector}`。
- `self.adata`：AnnData 对象（扰动数据，`adata.obs['condition']` 为扰动条件）。
- `self.mean_expression`：对照组（默认 `ctrl`）平均表达，用于中心化评价。

核心方法：

1. `load_dataset(dataset_path)`
   - 使用 `scanpy.read_h5ad` 读取数据。
   - 计算对照平均表达（`get_mean_control`）。

2. `get_mean_control(control_label='ctrl')`
   - 计算对照组均值表达向量，作为中心化基准。

3. `populate_X_y(mean_dict, X, y, embedding_size, interaction=False)`
   - 输入：每个条件的平均表达（`mean_dict`）。
   - 输出：构造回归特征矩阵 `X` 和目标矩阵 `y`。
   - 规则：
     - 单基因扰动：直接用该基因 embedding（归一化）。
     - 多基因扰动：对多个 embedding 求和后归一化（可选加入交互项）。
     - 如果基因名不在 embedding 字典中，用随机向量代替。

4. `evaluate_performance_rowwise(y_true, y_pred, mean_expression=None)`
   - 对每个扰动条件逐行评估：
     - **RMSE**
     - **MSE**
     - **MAE**
     - **Pearson 相关**
   - 注意：评估前会对预测和真实表达 **减去对照均值**（评估扰动效应）。

5. `run_experiment_with_conditions(...)`
   - 输入：训练条件、测试条件（condition list）
   - 流程：
     1) 根据 `condition` 拆分训练/测试数据
     2) 使用 `populate_dicts` 汇总每个条件的平均表达
     3) 构造 `X_train / y_train / X_test / y_test`
     4) 依次评估模型：
        - **train_condition baseline**（均值/中位数）
        - **Ridge Regression**
        - **KNN Regression**
        - **MLP**（`use_mlp=True` 时）
     5) 保存每个模型的：
        - 汇总指标（均值）
        - 每个基因的预测结果（`per_gene`）

6. `run_experiment_with_adata(adata_train, adata_test, ...)`
   - 与上一个类似，但输入直接是拆分好的 AnnData。

7. `run_kfold_experiments(...)`
   - 对条件集（`unique_conditions`）做 KFold 切分。
   - 每个 fold 调用 `run_experiment_with_conditions`。
   - 汇总各模型指标的均值与标准差。

---

### 2) 其他辅助函数

#### `calculate_distances(X_train, X_test)`
计算测试样本在 embedding 空间中与训练样本的余弦距离：
- `closest_distances`
- `avg_top10_distances`
用于结果分析（和 per-gene 输出绑定）。

#### `TrainConditionModel`
**基线模型**：只返回训练集表达的均值/中位数。
- `fit`：计算均值或中位数
- `predict`：对所有测试条件输出同一个常数向量

#### `populate_dicts(adata_subset, mean_dict)`
把 AnnData 子集按 `condition` 分组，计算每个条件的平均表达，放入字典。

---

### 3) 关键调用链（主流程）

以 notebook 为例：

1) 读取数据 `scanpy.read_h5ad`  → `GenePertExperiment.adata`
2) 读取 embedding → `GenePertExperiment.embeddings`
3) 调用 `run_experiments_with_embeddings(...)`（在 `utils.py`）
4) 进入 `GenePertExperiment.run_kfold_experiments(...)`
5) KFold 内循环调用 `run_experiment_with_conditions(...)`
6) 每个模型（Ridge/KNN/MLP/基线）训练 + `evaluate_performance_rowwise`
7) 汇总指标 → 返回给 `utils.py`
8) `plot_mse_corr_comparison(...)` 绘图

---

## `utils.py` 的作用与函数

### 1) `run_experiments_with_embeddings(...)`
- 作用：对多个 embedding 文件循环跑实验，汇总结果。
- 内部调用：
  - `GenePertExperiment.run_kfold_experiments(...)`
  - `get_best_overall_mse_corr(...)`
  - `get_ranked_genes_by_correlation(...)`
  - `get_gene_predictions(...)`
- 输出：`results_comparison`，用于绘图与对比。

### 2) `plot_mse_corr_comparison(...)`
- 画 3 个子图：**MSE / RMSE / Pearson**
- 依据 `results_comparison` 中的 best model 统计结果绘制。

### 3) `compare_embedding_correlations(...)`
- 比较不同 embedding 的相关性散点图。

---

## notebooks 与脚本的调用方式

### Notebook（Adamson / Norman）
- 加载数据 + obs
- 指定 embedding 文件
- 调用 `run_experiments_with_embeddings(...)`
- 绘制 MSE / RMSE / Pearson 图

### 脚本（`run_genepert_biolord.py`）
- `run_adamson()` / `run_norman()`
- 不依赖 notebook，适合批量跑实验

---

## 常见问题与注意点

1) **embedding 覆盖率**
   - 若条件中的基因不在 embedding 中，会用随机向量补充，影响性能。

2) **评估指标**
   - RMSE / MSE / Pearson 都是基于 “扰动效应”（表达减去对照均值）计算。

3) **use_mlp=True**
   - 需要 PyTorch 正常安装；否则会报 DLL 相关错误。

---

## 快速理解（核心一句话）

GenePert 的核心就是：
> 用基因的文本 embedding 作为扰动特征，回归预测扰动后基因表达变化，并在多数据集上用 Ridge/KNN/MLP 做对比评估。
