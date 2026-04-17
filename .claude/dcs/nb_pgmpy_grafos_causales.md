# Notebooks del Profesor — pgmpy: Descubrimiento Causal de Grafos

**Fuente:** `docs/src/VALERO_pgmpy_Structure Learning_adut_proc_simple.ipynb`  
**Fuente:** `docs/src/VALERO_pgmpy_Structure Learning_Bank.ipynb`

---

## Librería principal: pgmpy

```python
!pip install pgmpy
!apt install libgraphviz-dev
!pip install pygraphviz
```

> **AVISO:** En pgmpy >= 1.1.0 hay FutureWarnings — las clases se han movido de módulo:
> - `pgmpy.estimators.PC` → `pgmpy.causal_discovery.PC`
> - `pgmpy.estimators.HillClimbSearch` → `pgmpy.causal_discovery.HillClimbSearch`
> - `pillai_trace` → `pgmpy.ci_tests.PillaiTrace`
>
> El código del profesor usa la API antigua (sigue funcionando pero lanza warnings).

---

## Algoritmo PC (Peter-Clark)

```python
from pgmpy.estimators import PC  # o pgmpy.causal_discovery.PC

est_pc = PC(df)

# Variante 1: con test de Pillai (para datos mixtos/categóricos ordenados)
cpdag = est_pc.estimate(ci_test="pillai", return_type="cpdag")

# Variante 2: stable variant con max_cond_vars
estimated_model = est_pc.estimate(variant="stable", max_cond_vars=4)
```

**Parámetros clave:**
- `ci_test`: test de independencia condicional usado. Opciones: `"pillai"`, `"chi_square"`, `"pearsonr"`, `"g_sq"`
- `variant`: `"stable"` (recomendado, orden-independiente), `"original"`
- `max_cond_vars`: máximo número de variables condicionantes (controla complejidad)
- `return_type`: `"cpdag"` (grafo parcialmente dirigido), `"dag"`, `"pdag"`

**Output:** CPDAG (Completed Partially Directed Acyclic Graph) — algunas aristas pueden quedar sin orientar.

---

## Algoritmo Hill Climb Search (score-based)

```python
from pgmpy.estimators import HillClimbSearch

est_hill = HillClimbSearch(df)
dag_hill = est_hill.estimate(scoring_method="bic-d")
```

**Parámetros:**
- `scoring_method`: `"bic-d"` (BIC para datos discretos), `"k2"`, `"bdeu"`, `"bic-g"` (gaussiano)

**Output:** DAG completo con todas las aristas orientadas.

---

## Preprocesamiento de datos — patrón del profesor

El profesor convierte variables a `pd.Categorical` antes de pasar a pgmpy:

```python
# Variables ordinales (con orden)
df.Age = pd.Categorical(
    df.Age,
    categories=["<21", "21-30", "31-40", "41-50", "51-60", "61-70", ">70"],
    ordered=True,
)
df.Education = pd.Categorical(df.Education, categories=[...], ordered=True)

# Variables nominales (sin orden)
df.Workclass = pd.Categorical(df.Workclass, ordered=False)
df.Sex = pd.Categorical(df.Sex, ordered=False)
```

> **Para finanzas:** discretizar los retornos en categorías ("negativo", "neutro", "positivo") antes de usar PC con chi_square o pillai.

---

## Visualización del grafo

### Con graphviz (método del profesor)

```python
from IPython.display import Image

# Guardar y mostrar
model.to_graphviz().draw("output.png", prog="dot")   # layout jerárquico
# o
model.to_graphviz().draw("output.png", prog="neato")  # layout spring

Image("output.png")
```

### Con networkx (alternativa)

```python
import networkx as nx
import matplotlib.pyplot as plt

# Extraer matriz de adyacencia
nodes = estimated_model.nodes()

# Sin flechas (grafo no dirigido)
adj_undirected = nx.to_numpy_array(estimated_model.to_undirected(), nodelist=nodes, weight=None)

# Con flechas (grafo dirigido)
adj_directed = nx.to_numpy_array(estimated_model.to_directed(), nodelist=nodes, weight=None)

# Visualizar como heatmap
plt.imshow(adj_directed)
plt.show()
```

---

## Datasets usados por el profesor

| Dataset | Carga | Tipo |
|---------|-------|------|
| Adult Income (procesado) | `pd.read_csv(url_github_pgmpy)` | Tabular categórico |
| Bank Binary | `pd.read_excel("bank_binary.xlsx")` | Tabular mixto |
| **Acciones (taller B3-T3)** | `yf.download(tickers, start, end)['Adj Close']` | Series temporales financieras |
| **Factores Fama-French** | CSV descargado de Kenneth French / Dartmouth | Factores: Mkt-RF, SMB, HML, RMW, CMA |

URL del Adult procesado:
```
https://raw.githubusercontent.com/pgmpy/pgmpy/refs/heads/dev/pgmpy/tests/test_estimators/testdata/adult_proc.csv
```

### Flujo completo para datos financieros (demostrado en clase)

```python
import yfinance as yf
import pandas as pd

# Tickers usados por Valero en clase
tickers = ['GE', 'KO', 'IBM', 'MCD', 'WMT', 'BRK-B', '^GSPC']
data = yf.download(tickers, start='1980-01-01', end='2023-12-31')['Adj Close']

# IMPORTANTE: trabajar siempre con retornos, no con precios
returns = data.pct_change().dropna()

# IMPORTANTE: limpiar nombres de columnas — pgmpy falla con guiones y símbolos
returns.columns = [c.replace('-', '').replace('^', '') for c in returns.columns]

# Filtrar por periodo temporal (ej. para comparar épocas)
returns_old = returns['1980':'2000']
returns_new = returns['2001':'2023']
```

### Variable con Lag (para forzar causalidad temporal)

```python
# Añadir variable desplazada un día hacia adelante
df_with_lag = returns.copy()
df_with_lag['GE_lag'] = df_with_lag['GE'].shift(-1)
df_with_lag = df_with_lag.dropna()

# Resultado esperado: la flecha GE → GE_lag debe salir siempre
# (el valor de hoy predice el valor de mañana)
```

---

## Comparación PC vs HillClimb

| Aspecto | PC (constraint-based) | HillClimbSearch (score-based) |
|---------|----------------------|-------------------------------|
| Base | Tests de independencia condicional | Maximización de score (BIC) |
| Output | CPDAG (puede tener aristas no orientadas) | DAG completo |
| Velocidad | Más lento con muchas variables | Más rápido |
| Parámetro clave | `ci_test`, `max_cond_vars` | `scoring_method` |
| Uso prof. | Ambos, comparando resultados | sí |
| **Opinión Valero** | Válido; más flechas bidireccionales | **Preferido para series temporales** — grafos con "más lógica" |

> Valero dijo en clase: "el HC me da mejor pinta para datos financieros, tiene más lógica".

---

## Patrón completo del profesor (flujo estándar)

```python
import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt
from IPython.display import Image
from pgmpy.estimators import PC, HillClimbSearch

# 1. Cargar y preprocesar datos
df = pd.read_csv(...)
# Convertir a Categorical si procede

# 2. Aprender estructura con PC
est = PC(data=df)
model_pc = est.estimate(variant="stable", max_cond_vars=4)

# 3. Aprender estructura con HillClimb
est_hill = HillClimbSearch(df)
model_hill = est_hill.estimate(scoring_method="bic-d")

# 4. Visualizar
model_pc.to_graphviz().draw("pc_graph.png", prog="dot")
Image("pc_graph.png")

# 5. Extraer matriz de adyacencia
adj = nx.to_numpy_array(model_pc.to_directed(), nodelist=model_pc.nodes(), weight=None)
plt.imshow(adj)
```
