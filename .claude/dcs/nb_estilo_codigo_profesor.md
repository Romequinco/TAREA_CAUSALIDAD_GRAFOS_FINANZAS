# Estilo de Código del Profesor — Patrones Extraídos de los Notebooks

**Fuente:** todos los notebooks en `docs/src/`

---

## Principios generales observados

1. **Código muy conciso** — sin clases innecesarias, todo en celdas cortas y directas
2. **Comentarios en español en los markdowns, código en inglés**
3. **Siempre comparar baseline vs. método** — primero entrena un modelo simple de referencia
4. **Visualización inmediata** — `plt.plot(...)` después de cada resultado relevante
5. **Sin over-engineering** — no usa funciones auxiliares si no es necesario; el flujo es lineal celda a celda
6. **`random_state=0`** para reproducibilidad en los modelos que lo admiten

---

## Estructura típica de un notebook del profesor

```
1. Docstring de una línea con el propósito (en celda de código como string literal)
2. Imports
3. Carga y preparación de datos
4. Baseline (modelo simple para referencia)
5. Método principal
6. Evaluación + comparación con baseline
7. Visualización de resultados
```

---

## Imports habituales

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# sklearn
from sklearn.model_selection import train_test_split, KFold
from sklearn.utils import resample
from sklearn import metrics
from sklearn.tree import DecisionTreeRegressor
from sklearn.linear_model import LinearRegression, BayesianRidge, Lasso
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVR

# scipy
from scipy.stats import mode as moda

# pgmpy (grafos causales)
from pgmpy.estimators import PC, HillClimbSearch

# networkx
import networkx as nx

# visualización de grafos
from IPython.display import Image
import pygraphviz  # requiere: apt install libgraphviz-dev && pip install pygraphviz
```

---

## Patrones de evaluación

```python
# Métrica estándar
report = metrics.mean_squared_error

# Patrón comparación baseline vs método
print('Res tree:', report(yts, yp_baseline))
print('Res RF:  ', report(yts, yf))
print('Res OOB: ', report(ytr, yf_oob))
```

El profesor siempre reporta las métricas en la misma celda para facilitar la comparación visual.

---

## Bagging manual (patrón del profesor)

```python
T = 200  # número de modelos
bag = []

for t in range(T):
    xx, yy = resample(Xtr, ytr, replace=True)  # bootstrap
    pred = DecisionTreeRegressor(max_depth=max_depth)
    pred.fit(xx, yy)
    bag.append(pred)

# Predicción: media de todos los árboles
yb = np.zeros((T, len(yts)))
for n, t in enumerate(bag):
    yb[n, :] = t.predict(Xts)

yf = np.mean(yb, axis=0)  # agregación por media
```

---

## Random Forest manual con OOB (patrón del profesor)

```python
n_samples = np.arange(len(ytr))
bag = []
y_oob = np.ones((T, len(ytr))) * np.nan  # NaN donde no hay predicción OOB

for t in range(T):
    idx = resample(n_samples, replace=True)
    pred = DecisionTreeRegressor(max_depth=max_depth, max_features=0.5)  # RF: subconjunto de features
    pred.fit(Xtr[idx], ytr[idx])
    bag.append(pred)

    oob = np.setdiff1d(n_samples, idx)  # muestras no usadas en este árbol
    y_oob[t, oob] = pred.predict(Xtr[oob])

yf_oob = np.nanmean(y_oob, axis=0)  # media ignorando NaN
```

**Diferencia bagging vs RF:** RF añade `max_features=0.5` (aleatorización de features en cada árbol).

---

## Stacking / Aggregating (patrón del profesor)

```python
kf = KFold()
y_hat_all = np.zeros((len(ytr), T))  # predicciones out-of-fold

for itrain, itest in kf.split(Xtr):
    for t, c in enumerate(rlev1):
        c.fit(Xtr[itrain], ytr[itrain])
        y_hat_all[itest, t] = c.predict(Xtr[itest])

# Agregación: media, mediana, o media ponderada por inverse error
yts_mean   = np.mean(yts_hat, axis=1)
yts_median = np.median(yts_hat, axis=1)

# Media ponderada: peso = 1/error
E = [np.mean((yts - yts_hat[:, t])**2) for t in range(T)]
w = [(1/e) / sum(1/e_ for e_ in E) for e in E]
```

---

## LIME y SHAP (patrón del profesor)

```python
from lime.lime_tabular import LimeTabularExplainer
import shap

# LIME
explainer_lime = LimeTabularExplainer(
    Xtr.values,
    feature_names=X.columns.tolist(),
    class_names=data.target_names.tolist(),
    mode='classification'
)
exp = explainer_lime.explain_instance(x_instance, model.predict_proba, num_features=10)

# SHAP (Kernel SHAP — model agnostic)
background = shap.sample(Xtr, 100)  # dataset de referencia pequeño
explainer = shap.Explainer(model.predict_proba, background)
shap_values = explainer(Xts.iloc[:10])
shap.plots.waterfall(shap_values[0, :, 1])  # explicación local
```

---

## Visualización de grafos causales (patrón del profesor)

```python
# Opción 1: graphviz (la que usa el profesor — más bonita)
model.to_graphviz().draw("graph.png", prog="dot")   # jerárquico
model.to_graphviz().draw("graph.png", prog="neato") # spring layout
Image("graph.png")

# Opción 2: networkx + matplotlib (para matriz de adyacencia)
adj = nx.to_numpy_array(model.to_directed(), nodelist=model.nodes(), weight=None)
plt.imshow(adj)
plt.colorbar()
```

---

## Qué NO hace el profesor

- No usa `argparse`, clases propias, ni módulos externos complejos
- No separa el código en múltiples ficheros `.py`
- No hace logging ni manejo de excepciones
- No usa `tqdm` (deja que pgmpy muestre su propia barra de progreso)
- No normaliza/escala datos a menos que el método lo requiera explícitamente
- No genera informes automáticos — todo es visual e interactivo
