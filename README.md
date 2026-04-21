# Causalidad entre Variables Financieras con Grafos

Taller B3-T3 — Descubrimiento de estructura causal sobre sectores bursátiles de EE.UU. usando DAGs.

## Objetivo

Demostrar la diferencia entre correlacion (asociacion simetrica) y causalidad (relacion dirigida)
aplicada a sectores bursatiles, usando Grafos Acíclicos Dirigidos (DAG) aprendidos de datos historicos.

## Entregables

- **Notebook:** `notebook_causalidad.ipynb`
- **Explicacion:** `EXPLICACION_SENCILLA.md` — descripcion sin tecnicismos para cualquier lector
- **Video:** explicacion de 5 minutos, celda a celda, con interpretacion financiera

## Dataset

ETFs sectoriales de EE.UU. descargados via `yfinance` (2000-2024):

| Ticker | Nombre | Descripcion |
|---|---|---|
| XLF | Financiero | ETF sector financiero (bancos, aseguradoras) |
| XLE | Energia | ETF sector energético (petroleras, gasistas) |
| XLK | Tecnologia | ETF sector tecnológico (software, hardware, semis) |
| XLV | Salud | ETF sector salud (farma, hospitales, medtech) |
| XLI | Industrial | ETF sector industrial (manufactura, transporte, defensa) |
| XLY | Consumo | ETF consumo discrecional (retail, ocio, autos) |

## Estructura del Notebook

| Seccion | Contenido |
|---|---|
| 1. Datos y Correlacion | Descarga de retornos + matriz de Pearson en una celda |
| 2. DAG — Algoritmo PC | PC sobre datos completos, grafo dirigido y matriz de adyacencia |
| 3. Estabilidad temporal | 4 periodos en figura 2x2: pre/post-crisis y bull/bear market |
| 4. Comparacion de algoritmos | PC vs HillClimbSearch vs LiNGAM en figura 1x3 |
| 5. Causalidad temporal | Granger p-values y PC con lag-1 en figura conjunta |
| 6. Robustez del PC | Sensibilidad al alpha y bootstrap en figura conjunta |
| 7. Visualizacion interactiva | DAG con flechas dirigidas, zoom y hover (plotly, inline) |
| 8. Factor Mirage | Collider bias: controlar por un colisionador induce correlacion espuria |

## Algoritmos

- **PC** (`pgmpy.estimators.PC`): elimina aristas por tests de independencia condicional (`pearsonr`), orienta colisionadores. Parametros: `variant='stable'`, `significance_level=0.05`.
- **HillClimbSearch** (`pgmpy.estimators.HillClimbSearch`): maximiza BIC Gaussiano de forma greedy. Parametros: `scoring_method='bic-g'`.
- **LiNGAM** (`lingam.DirectLiNGAM`): usa ICA para explotar la no-gaussianidad de los retornos y orientar completamente todas las aristas.
- **Granger** (`statsmodels`): test de precedencia temporal, `maxlag=5`.

## Instalacion

```bash
pip install pgmpy yfinance networkx matplotlib pandas numpy scikit-learn seaborn lingam statsmodels plotly
```

## Uso

Abrir `notebook_causalidad.ipynb` y ejecutar `Kernel > Restart & Run All`.

Si el algoritmo PC devuelve un grafo vacio, subir `significance_level` de `0.05` a `0.10`.

## Referencia

Lopez de Prado, M. & Zoonekynd, V. (2025). *Causality and Factor Investing: A Primer*.
Finance, Economics and Banking (FEB-RN).
