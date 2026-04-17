# Contexto: Taller B3-T3 — Causalidad entre Variables Financieras con Grafos

## Metadatos
- **Entrega:** 23 de Abril de 2026, 23:59 (aula virtual) ← cambiado por Valero en clase
- **Formato entregable:** Un Notebook Jupyter/Colab + vídeo de 5 minutos
- **Grupo:** 2 estudiantes

---

## Objetivo académico

Demostrar que se entiende la diferencia entre **correlación** (asociación estadística) y **causalidad** (relación dirigida causa-efecto) aplicada a variables financieras, usando **Grafos Acíclicos Dirigidos (DAG)**.

---

## Lo que hay que construir (Notebook mínimo)

| Paso | Tarea | Notas técnicas |
|------|-------|----------------|
| 1 | **Carga de datos** | Dataset financiero a elegir (acciones, factores, macro...) |
| 2 | **Matriz de correlaciones** | Pearson estándar, simétrica, sin dirección |
| 3 | **Matriz direccional** | Asimétrica — cuantifica quién "lidera" a quién (p.ej. via cross-correlación con lag o test de Granger) |
| 4 | **Grafos causales (DAG)** | Usar librería de descubrimiento causal (p.ej. `causallearn`, `castle`, `tigramite`) |
| 5 | **Factor Mirage** *(extra)* | Analizar correlaciones espurias que desaparecen al controlar por una variable común |

---

## Análisis comparativo de grafos (punto 4 del enunciado)

Hay que mostrar cómo el DAG cambia según:

- **Periodos temporales** → comparar grafo en distintas ventanas (p.ej. pre/post crisis, bull vs bear market)
- **Con y sin LAG** → incluir retardos temporales en las variables para capturar causalidad temporal (Granger)
- **Métodos de ajuste** → probar distintos algoritmos: PC, GES, LiNGAM, NOTEARS...
- **Parámetros del modelo** → umbral de significancia (alpha), número de lags, test estadístico usado
- **Semilla** → fijar/variar `random_state` para ver estabilidad del grafo
- **Tipos de visualización** → networkx, graphviz, pyvis, matriz de adyacencia...

---

## Diferencia clave: Correlación vs Causalidad

```
Correlación:  A ↔ B  (simétrica, no dice quién causa a quién)
Causalidad:   A → B  (dirigida, implica que A precede/determina B)
```

**Ejemplo financiero:** El VIX y las caídas del S&P500 están correlacionados, pero el DAG debe revelar si el VIX *causa* las caídas o simplemente las acompaña.

---

## Dataset recomendado

Opciones interesantes:
- **Factores Fama-French** (SMB, HML, Mkt-RF, RMW, CMA) — muy usado en factor investing
- **ETFs sectoriales** (XLF, XLE, XLK...) — relaciones macro entre sectores
- **Macro variables** (tipos de interés, inflación CPI, VIX, DXY, precio del petróleo)
- **Criptomonedas** (BTC, ETH, correlaciones con activos tradicionales)

> Recomendación: **Factores Fama-French + VIX + tipos** — bien documentados, fácil de descargar con `yfinance` o `pandas_datareader`, y tienen relaciones causales conocidas en la literatura.

---

## Librerías Python (confirmadas por Valero en clase)

```python
# Descarga de datos
yfinance             # datos financieros históricos (Yahoo Finance)
pandas_datareader    # Fama-French y otros

# Análisis causal — LIBRERÍA PRINCIPAL DEL PROFESOR
pgmpy                # PC + HillClimbSearch. La más estable/robusta según Valero
# !pip install pgmpy
# !apt install libgraphviz-dev && pip install pygraphviz  (para draw())

# Visualización
networkx, matplotlib
graphviz / pygraphviz  # para model.to_graphviz().draw(...)
```

> **Nota:** Valero descartó explícitamente `causallearn`, `castle` y `tigramite` para este taller. La librería usada en clase es únicamente `pgmpy`.

---

## Estructura del Notebook (validada por Valero en clase)

Valero insistió en que sea **lo más mínimo posible** — si cabe en 10 líneas, mejor que en 100.

```
1. Instalación pgmpy (1 celda)
2. Imports
3. Carga de datos financieros + conversión a retornos + limpieza de nombres de columnas
4. Matriz de correlaciones → heatmap
5. Estimación del DAG (PC o HillClimbSearch)
6. Matriz direccional (adyacencia) + visualización del grafo
7. Variación 1: [elegir una — ej. dos periodos temporales]
8. Variación 2: [elegir una — ej. cambiar algoritmo o significance_level]
9. [EXTRA] Factor Mirage: demostrar que usar variables con flecha invertida
         sube el R² artificialmente pero rompe el modelo en producción
```

> No hace falta implementar todas las variaciones. Valero dijo explícitamente:
> elegir 1 o 2 variaciones y explicarlas bien es mejor que hacer 6 mal.

---

## Factor Mirage (extra)

Un **factor mirage** es una variable que parece tener poder predictivo/causal sobre los retornos, pero cuya relación desaparece cuando se controla correctamente por otras variables (es una correlación espuria). En el grafo DAG, esto se manifiesta como una arista que aparece con ciertos parámetros pero desaparece al introducir la variable confusora correcta.

Ejemplo clásico: momentum puede parecer causar retornos futuros, pero al controlar por calidad (QMJ) la relación se debilita.

---

## Criterios de evaluación

| Peso | Componente |
|------|------------|
| 30% | Notebook (código limpio, mínimo y funcional) |
| 70% | Explicación en vídeo (código línea a línea + interpretación financiera de los grafos) |

---

## Bibliografía base

> *"Causality and Factor Investing: A Primer"*  
> Marcos López de Prado, Vincent Zoonekynd  
> Finance, Economics and Banking (FEB-RN), 2025

---

## Notas de trabajo

- Código lo más conciso posible (el enunciado pide **mínimo número de líneas**)
- El vídeo es el 70% de la nota — dedicar tiempo a la explicación financiera, no solo técnica
- Fijar semillas para reproducibilidad
- Comentar cada celda brevemente para facilitar la explicación en vídeo
