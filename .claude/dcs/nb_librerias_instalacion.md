# Librerías del Proyecto — Instalación y Referencia

Extraído de los notebooks del profesor en `docs/src/`.

---

## Librerías core del proyecto (grafos causales + finanzas)

```bash
# Grafos causales
pip install pgmpy
pip install causallearn       # PC, FCI, GES, LiNGAM
pip install lingam            # LiNGAM standalone

# Visualización de grafos
apt install libgraphviz-dev   # sistema (Colab/Linux)
pip install pygraphviz
pip install networkx

# Datos financieros
pip install yfinance
pip install pandas_datareader

# ML estándar
pip install scikit-learn numpy pandas matplotlib scipy

# Explicabilidad (notebooks del profesor)
pip install lime shap
```

---

## pgmpy — API actual vs deprecated

| API antigua (profesor usa) | API nueva (pgmpy >= 1.1) |
|---------------------------|--------------------------|
| `pgmpy.estimators.PC` | `pgmpy.causal_discovery.PC` |
| `pgmpy.estimators.HillClimbSearch` | `pgmpy.causal_discovery.HillClimbSearch` |
| `ci_test="pillai"` (string) | `pgmpy.ci_tests.PillaiTrace` |

> El código del profesor funciona con ambas versiones (con FutureWarning). Para producción, usar la API nueva.

---

## Tests de independencia condicional disponibles en pgmpy

| Test | `ci_test=` | Tipo de datos |
|------|-----------|---------------|
| Chi-cuadrado | `"chi_square"` | Categóricos |
| G-test | `"g_sq"` | Categóricos |
| Pearson r | `"pearsonr"` | Continuos, relaciones lineales |
| Pillai trace | `"pillai"` | Mixtos / multivariante |

> Para datos financieros continuos: usar `"pearsonr"` (lineal) o discretizar y usar `"chi_square"`.

---

## Scoring methods en HillClimbSearch

| Score | `scoring_method=` | Tipo de datos |
|-------|------------------|---------------|
| BIC discreto | `"bic-d"` | Categóricos |
| BIC gaussiano | `"bic-g"` | Continuos |
| K2 | `"k2"` | Categóricos |
| BDeu | `"bdeu"` | Categóricos (Bayesiano) |

> Para retornos financieros continuos: usar `"bic-g"`.

---

## causallearn — alternativa a pgmpy

```python
from causallearn.search.ConstraintBased.PC import pc
from causallearn.search.ScoreBased.GES import ges
from causallearn.search.FCMBased import lingam

# PC
cg = pc(data_array, alpha=0.05, indep_test='fisherz')

# GES
record = ges(data_array)

# LiNGAM
model = lingam.ICALiNGAM()
model.fit(data_array)
print(model.adjacency_matrix_)
```

---

## Visualización: opciones y cuándo usar cada una

| Herramienta | Cuándo | Cómo |
|-------------|--------|------|
| `model.to_graphviz().draw(...)` | Grafos causales pgmpy — más bonito | Requiere pygraphviz |
| `networkx` + `matplotlib` | Matrices de adyacencia, grafos simples | Sin dependencias externas |
| `pyvis` | Grafos interactivos en HTML | `pip install pyvis` |
| `plt.imshow(adj)` | Visualizar matriz de adyacencia como heatmap | Solo numpy/matplotlib |

---

## Datasets financieros — carga estándar

```python
import yfinance as yf
import pandas as pd

# Múltiples activos
tickers = ["SPY", "TLT", "GLD", "USO", "^VIX"]
data = yf.download(tickers, start="2018-01-01", end="2023-12-31")["Adj Close"]
returns = data.pct_change().dropna()

# Factores Fama-French
import pandas_datareader.data as web
ff = web.DataReader("F-F_Research_Data_5_Factors_2x3_daily", "famafrench", start="2018")[0]
ff = ff / 100  # convertir de % a decimal
```
