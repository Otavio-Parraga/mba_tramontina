# Roteiro 2 — Treinamento de Modelos no Orange

**Disciplina:** Machine Learning · Dia 2
**Bases:** `Dados/tramontina_clientes.csv` (classificação) · `Dados/tramontina_clustering.csv` (clustering)
**Referência:** `Notebooks/02_Treinamento_de_Modelos.ipynb`

Monte dois fluxos no Orange Data Mining: **Parte A** (classificação de churn) e **Parte B** (segmentação com k-Means).

---

## Parte A — Classificação de Churn

### A.0 Carregamento
1. Adicione **File** e carregue `Dados/tramontina_clientes.csv`.
2. Em *Columns*:
   - `ID_Cliente` → **meta**
   - `Churn` → **target**
   - Demais colunas → **feature**

### A.1 Impute
1. Adicione **Impute** e conecte `File → Impute`.
2. *Default method*: **Average/Most frequent**.
3. Cobertura para `Desconto_Medio_%` e `Satisfacao_NPS_1a10`.

### A.2 Preprocess
1. Adicione **Preprocess** e conecte `Impute → Preprocess`.
2. Métodos, nesta ordem:
   - **Continuize Discrete Variables** → *One attribute per value* (`Regiao`, `Canal_Venda`, `Segmento`).
   - **Normalize Features** → *Standardize to μ=0, σ²=1*.

### A.3 Modelos (learners)
Adicione os quatro widgets de aprendizado (não precisam de conexão com dados; serão ligados ao Test and Score):

| Widget | Parâmetro |
|--------|-----------|
| **kNN** | *Number of neighbors* = 5 |
| **Tree** | *Limit the maximal tree depth* = 5 |
| **Logistic Regression** | *Regularization* = Ridge (L2) |
| **Random Forest** | *Number of trees* = 100 |

### A.4 Test and Score
1. Adicione **Test and Score**.
2. Conecte `Preprocess → Test and Score` (Data).
3. Conecte cada learner ao **Test and Score**: `kNN → Test and Score`, `Tree → Test and Score`, `Logistic Regression → Test and Score`, `Random Forest → Test and Score`.
4. *Sampling*: **Cross validation**, *Number of folds* = 5, **Stratified**.
5. *Target class*: `Sim`.
6. Compare as colunas **AUC, CA, F1, Precision, Recall**.

### A.5 Matriz de confusão
1. Adicione **Confusion Matrix** e conecte `Test and Score → Confusion Matrix`.
2. Selecione um modelo e observe Verdadeiros/Falsos Positivos e Negativos.

### A.6 Curva ROC
1. Adicione **ROC Analysis** e conecte `Test and Score → ROC Analysis`.
2. *Target*: `Sim`.
3. Compare as curvas dos quatro modelos.

### A.7 Ranking de atributos (opcional)
1. Adicione **Rank** e conecte `Preprocess → Rank`.
2. Ordene por *Gain ratio* ou *Gini* para ver os atributos mais associados ao churn.

---

## Parte B — Segmentação com k-Means

### B.0 Carregamento
1. Adicione um novo **File** e carregue `Dados/tramontina_clustering.csv`.
2. Em *Columns*:
   - `ID_Cliente` → **meta**
   - `Canal_Predominante` → **meta**
   - Demais colunas → **feature**

### B.1 Select Columns
1. Adicione **Select Columns** e conecte `File → Select Columns`.
2. Mantenha como *features*:
   - `Gasto_Mensal_R$`
   - `Itens_por_Pedido`
   - `Frequencia_Compras_mes`
   - `Perc_Linha_Cozinha_Profissional`
   - `Anos_como_Cliente`

### B.2 Preprocess
1. Adicione **Preprocess** e conecte `Select Columns → Preprocess`.
2. Método: **Normalize Features** → *Standardize to μ=0, σ²=1*.

### B.3 k-Means
1. Adicione **k-Means** e conecte `Preprocess → k-Means`.
2. *Number of clusters*: **Fixed = 3**.
3. Para comparar, marque *From* 2 *to* 8 e observe a coluna **Silhouette**.
4. *Initialization*: k-Means++ · *Re-runs*: 10.

### B.4 Silhouette Plot
1. Adicione **Silhouette Plot** e conecte `k-Means → Silhouette Plot`.
2. *Cluster variable*: `Cluster`.

### B.5 PCA + Scatter Plot
1. Adicione **PCA** e conecte `k-Means → PCA`.
2. *Components*: 2.
3. Adicione **Scatter Plot** e conecte `PCA → Scatter Plot`.
4. *Axis x*: PC1 · *Axis y*: PC2 · *Color*: `Cluster`.

### B.6 Perfil dos clusters
1. Adicione **Box Plot** e conecte `k-Means → Box Plot`.
2. *Subgroups*: `Cluster`.
3. Percorra as cinco variáveis para perfilar cada grupo.
4. (Opcional) **Data Table** ligada a `k-Means` para inspecionar o `Cluster` por cliente.

---

## Mapa de conexões

```
PARTE A — Classificação
File ──► Impute ──► Preprocess ──┬─► Test and Score ──┬─► Confusion Matrix
                                 │                     └─► ROC Analysis
                                 └─► Rank
kNN ─────────────► Test and Score
Tree ────────────► Test and Score
Logistic Regression ► Test and Score
Random Forest ───► Test and Score

PARTE B — Clustering
File (2) ──► Select Columns ──► Preprocess ──► k-Means ──┬─► Silhouette Plot
                                                         ├─► PCA ──► Scatter Plot
                                                         └─► Box Plot
```
