# Roteiro 1 — Análise e Processamento de Dados no Orange

**Disciplina:** Data Mining · Dia 1
**Base:** `Dados/tramontina_clientes.csv`
**Referência:** `Notebooks/01_Analise_e_Processamento_de_Dados.ipynb`

Reproduza no Orange Data Mining o pipeline da imagem. Cada bloco abaixo indica o widget, a conexão e os parâmetros.

---

## 0. Preparação

1. Abra o Orange Data Mining.
2. Arraste o widget **File** para o canvas.
3. No **File**, carregue `Dados/tramontina_clientes.csv`.
4. Em *Columns*, ajuste os papéis:
   - `ID_Cliente` → **meta**
   - `Churn` → **target**
   - Demais colunas → **feature**

---

## 1. Exploração (ramo direto do File)

Conecte todos os widgets desta seção diretamente à saída **Data** do **File**.

### 1.1 Data Table
1. Adicione **Data Table** e conecte `File → Data Table`.
2. Abra e confira as 25 colunas e o nº de linhas.

### 1.2 Column Statistics
1. Adicione **Column Statistics** e conecte `File → Column Statistics`.
2. Observe média, mediana, mínimo, máximo e dispersão das colunas numéricas.

### 1.3 Distributions
1. Adicione **Distributions** e conecte `File → Distributions`.
2. *Variable*: `Faturamento_Anual_R$`.
3. *Split by*: `Churn`.
4. *Bins*: 30.

### 1.4 Box Plot
1. Adicione **Box Plot** e conecte `File → Box Plot`.
2. *Variable*: `Faturamento_Anual_R$`.
3. *Subgroups*: `Canal_Venda`.
4. Anote o **limite superior** (bigode direito / Q3 + 1,5·IQR) — será usado no Select Rows.

### 1.5 Scatter Plot
1. Adicione **Scatter Plot** e conecte `File → Scatter Plot`.
2. *Axis x*: `Ticket_Medio_R$`.
3. *Axis y*: `Faturamento_Anual_R$`.
4. *Color*: `Churn`.

### 1.6 Correlations
1. Adicione **Correlations** e conecte `File → Correlations`.
2. Método: **Pearson**.
3. Ordene pelo valor absoluto da correlação.

### 1.7 Formula
1. Adicione **Formula** (Feature Constructor) e conecte `File → Formula`.
2. Crie os atributos:

   | Nome | Tipo | Expressão |
   |------|------|-----------|
   | `Faturamento_por_Pedido` | Numeric | `Faturamento_Anual_R$ / Frequencia_pedidos_ano if Frequencia_pedidos_ano > 0 else 0` |
   | `Total_Categorias_Compradas` | Numeric | `Comprou_Cutelaria + Comprou_Panelas + Comprou_Utilidades_Cozinha + Comprou_Ferramentas + Comprou_Eletroportateis + Comprou_Moveis + Comprou_Talheres + Comprou_Jardinagem` |
   | `Cliente_Insatisfeito` | Categorical | `1 if Satisfacao_NPS_1a10 <= 6 else 0` |
   | `Porte_Cliente_ord` | Numeric | `1 if Porte_Cliente == "Pequeno" else 2 if Porte_Cliente == "Médio" else 3` |

   > Selecione cada variável pelo botão de seleção do widget para inserir o nome correto (colunas com `$` e `%` são referenciadas pelo seletor).

---

## 2. Processamento (pipeline)

### 2.1 Impute
1. Adicione **Impute** e conecte `File → Impute`.
2. *Default method*: **Average/Most frequent**.
3. Em *Individual settings*, defina para as colunas com valores ausentes:
   - `Desconto_Medio_%`
   - `Satisfacao_NPS_1a10`

### 2.2 Select Rows
1. Adicione **Select Rows** e conecte `Impute → Select Rows`.
2. Adicione a condição:
   - `Faturamento_Anual_R$` · *is below* · **[limite superior anotado no Box Plot]**
3. *Send automatically* ligado.

### 2.3 Preprocess
1. Adicione **Preprocess** e conecte `Select Rows → Preprocess`.
2. Adicione os métodos, nesta ordem:
   - **Continuize Discrete Variables** → *One attribute per value* (one-hot para `Regiao`, `Canal_Venda`, `Segmento`).
   - **Normalize Features** → *Standardize to μ=0, σ²=1*.

### 2.4 Data Table (1)
1. Adicione **Data Table** e conecte `Preprocess → Data Table (1)`.
2. Confira as colunas one-hot e os valores padronizados.

### 2.5 Formula (1)
1. Adicione **Formula** e conecte `Preprocess → Formula (1)`.
2. Crie os atributos derivados sobre os dados processados:

   | Nome | Tipo | Expressão |
   |------|------|-----------|
   | `Faturamento_por_Pedido` | Numeric | `Faturamento_Anual_R$ / Frequencia_pedidos_ano if Frequencia_pedidos_ano > 0 else 0` |
   | `Total_Categorias_Compradas` | Numeric | `Comprou_Cutelaria + Comprou_Panelas + Comprou_Utilidades_Cozinha + Comprou_Ferramentas + Comprou_Eletroportateis + Comprou_Moveis + Comprou_Talheres + Comprou_Jardinagem` |

### 2.6 Data Table (2)
1. Adicione **Data Table** e conecte `Formula (1) → Data Table (2)`.
2. Confira o conjunto final.

### 2.7 Save Data (exportação)
1. Adicione **Save Data** e conecte `Formula (1) → Save Data`.
2. Salve como `tramontina_clientes_processado.csv`.

---

## Mapa de conexões

```
File ──► Data Table
     ├─► Column Statistics
     ├─► Distributions
     ├─► Box Plot
     ├─► Scatter Plot
     ├─► Correlations
     ├─► Formula
     └─► Impute ──► Select Rows ──► Preprocess ──┬─► Data Table (1)
                                                 └─► Formula (1) ──┬─► Data Table (2)
                                                                   └─► Save Data
```
