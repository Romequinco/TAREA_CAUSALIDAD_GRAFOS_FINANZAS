# Causalidad entre Variables Financieras con Grafos

Taller B3-T3 — Descubrimiento de estructura causal sobre variables macro financieras usando DAGs.

## Objetivo

Demostrar la diferencia entre correlacion (asociacion simetrica) y causalidad (relacion dirigida)
aplicada a variables macro, usando Grafos Acíclicos Dirigidos (DAG) aprendidos de datos historicos.

## Entregable

- **Notebook:** `notebook_causalidad.ipynb`
- **Video:** explicacion de 5 minutos, celda a celda, con interpretacion financiera

## Dataset

Variables macro diarias descargadas via `yfinance` (2000-2024):

| Ticker | Nombre | Descripcion |
|---|---|---|
| ^GSPC | SP500 | Indice S&P 500 |
| ^VIX | VIX | Indice de volatilidad implícita |
| GLD | GLD | ETF de oro |
| TLT | TLT | ETF de bonos del Tesoro largo plazo |
| DX-Y.NYB | DXY | Indice del dolar americano |

## Estructura del Notebook

| Seccion | Contenido |
|---|---|
| 1. Correlacion | Matriz de Pearson — simetrica, sin direccion |
| 2. DAG (PC) | Algoritmo Peter-Clark sobre datos completos 2000-2024 |
| 3. Visualizacion | Grafo dirigido + matriz de adyacencia asimetrica |
| 4. Variacion 1 | Dos periodos: pre-crisis (2000-2007) vs post-crisis (2010-2023) |
| 5. Variacion 2 | PC (constraint-based) vs HillClimbSearch (score-based) |
| 6. Factor Mirage | Collider bias: controlar por un colisionador induce correlacion espuria |

## Algoritmos

- **PC** (`pgmpy.estimators.PC`): elimina aristas por tests de independencia condicional (`pearsonr`), orienta colisionadores. Parametros: `variant='stable'`, `significance_level=0.05`.
- **HillClimbSearch** (`pgmpy.estimators.HillClimbSearch`): maximiza BIC Gaussiano de forma greedy. Parametros: `scoring_method='bic-g'`.

## Instalacion

```bash
pip install pgmpy yfinance networkx matplotlib pandas numpy scikit-learn seaborn
```

## Uso

Abrir `notebook_causalidad.ipynb` y ejecutar `Kernel > Restart & Run All`.

Si la celda del algoritmo PC devuelve una lista vacia de aristas, subir `significance_level` de `0.05` a `0.10`.

## Referencia

Lopez de Prado, M. & Zoonekynd, V. (2025). *Causality and Factor Investing: A Primer*.
Finance, Economics and Banking (FEB-RN).
