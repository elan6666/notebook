# GenePT 生成流程笔记（中文）

本笔记**专门说明 GenePT 嵌入如何生成**，与 GenePert 预测模型解耦。

---

## 1. 生成目标

GenePT 的目标是得到一个 **基因名 → 向量** 的嵌入字典：
```
{ "GENE_SYMBOL": embedding_vector, ... }
```
该字典最终保存为 `.pickle` 文件，供 GenePert 使用。

---

## 2. 核心思路（两步）

### Step A：获取基因文本摘要
- 来源：NCBI Gene 页面（可选增加 UniProt 蛋白摘要）
- 结果：
  - `NCBI_summary_of_genes.json`
  - `NCBI_UniProt_summary_of_genes.json`

### Step B：把文本转成向量
- 使用 OpenAI 的文本嵌入模型：
  - `text-embedding-ada-002`（旧版，1536 维）
  - `text-embedding-3-large`（新版，3072 维）
- 结果：
  - `GenePT_gene_embedding_ada_text.pickle`
  - `GenePT_gene_protein_embedding_model_3_text.pickle`

---

## 3. GenePT 仓库里对应的 Notebook

在 `GenePT-main\GenePT-main` 中：

1) **`request_ncbi_text_for_genes.ipynb`**
   - 输入：基因名列表（例如从 Geneformer/scGPT 词表导出）
   - 输出：基因摘要 JSON（NCBI / UniProt）

2) **`gene_embeddings_examples.ipynb`**
   - 输入：摘要 JSON
   - 调用 OpenAI embedding API
   - 输出：`.pickle` 文件（gene → vector）

---

## 4. 生成流程示意（伪代码）

```python
# Step A: 获取基因摘要
for gene in gene_list:
    text = fetch_ncbi_summary(gene)
    save_json[gene] = text

# Step B: 文本转 embedding
for gene, text in save_json.items():
    emb = openai_embedding(text, model='text-embedding-3-large')
    embedding_dict[gene] = emb

# 保存
pickle.dump(embedding_dict, 'GenePT_gene_embedding.pickle')
```

---

## 5. 你当前已有的 GenePT 产物

你已经有生成好的文件（可直接用于 GenePert）：

- `data\GenePT_emebdding_v2\GenePT_gene_embedding_ada_text.pickle`
  - NCBI 文本 + `text-embedding-ada-002`

- `data\GenePT_emebdding_v2\GenePT_gene_protein_embedding_model_3_text.pickle`
  - NCBI + UniProt 文本 + `text-embedding-3-large`

---

## 6. 常见注意事项

- **基因名一致性**：GenePT 输出的 key 必须与数据集 `condition` 中的基因名一致。
- **缺失基因**：如果嵌入里没有该基因，GenePert 会用随机向量补。
- **维度差异**：不同 embedding 维度不同（1536 vs 3072），实验中不要混用。

---

## 7. 一句话总结

GenePT 本质就是：
> 把“基因文本摘要”送进文本嵌入模型，得到每个基因的向量表示，并保存成字典供下游使用。
