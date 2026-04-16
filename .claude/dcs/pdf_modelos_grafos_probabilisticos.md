# PDF: Modelos en Grafos Probabilísticos (Redes Bayesianas)

**Fuente:** `docs/raw/Modelos_de_Gafos_Probabilisticos_2025.pdf`  
**Autor:** Valero Laparra (mismo que Medidas de Dependencia)  
**Páginas:** 60

---

## Estructura del documento

1. Probabilidad — qué es, distribuciones
2. Probabilidad Bayesiana — Teorema de Bayes, ejercicios
3. Modelos Probabilísticos — distribuciones, convoluciones, MLE, KL divergence
4. Grafos — matriz de adyacencia, matriz Laplaciana
5. Modelos en Grafo — aprender estructura, parámetros, inferencia
6. Notación — convenciones de redes bayesianas

---

## Probabilidad Bayesiana

$$P(H|E) = \frac{P(E|H) \cdot P(H)}{P(E)}$$

**Prior no informativo (Bernardo):**
$$p = \frac{N_+ + 1}{N + 2}$$

Ejemplo del documento: test COVID con p(+|enfermo)=0.95, p(+|sano)=0.01 → resultado depende fuertemente del prior p(enfermo).

---

## Grafos — conceptos clave

**Matriz de adyacencia:** entrada (i,j) = 1 si hay arista de i a j.

**Matriz Laplaciana:**
$$L = D - A$$
Donde D es la matriz de grados y A la de adyacencia. Usada en análisis espectral de grafos.

---

## Aprender la Estructura del Grafo

El profesor enlista tres componentes del aprendizaje en grafos:

### 1. Aprender estructura
- Búsqueda Exhaustiva
- **Algoritmo PC** ← el principal
- Hill-Climbing
- Árbol

### 2. Aprender parámetros probabilísticos
- Maximum Likelihood
- Bayesian
- Divergencia KL

### 3. Hacer inferencia
- Montecarlo
- Sampling methods

---

## Algoritmo PC — Suposiciones

El profesor es explícito sobre las tres asunciones del PC:

1. **Acíclico** — el grafo es un DAG, sin ciclos
2. **Propiedad de Markov** — cada nodo es independiente de sus no-descendientes dado sus padres
3. **Completitud** — el proceso de independencia que generó los datos es fiel al grafo

> Referencia: https://towardsdatascience.com/causal-discovery-6858f9af6dcb

---

## Aprender estructura: conexión con medidas de dependencia

El documento conecta explícitamente el aprendizaje de grafos con las medidas de dependencia:

```
Aprender estructura está relacionado con:
- Probabilidad condicionada: P(X|Y)
- Medidas de independencia: I(X,Y)          ← Información Mutua
- Independencia condicionada: I(X,Y|Z)      ← MI Condicional
- Relaciones causales: X→Y vs Y→X          ← Granger, ANM
```

---

## Herramientas mencionadas por el profesor

| Herramienta | URL | Tipo |
|------------|-----|------|
| **pgmpy** | https://pgmpy.org | Python — principal |
| PyBNesian | https://github.com/davenza/PyBNesian | Python — alternativa |
| Bayes Server | https://online.bayesserver.com/ | GUI online |

### Flujo con Bayes Server (GUI)
1. File → New
2. Data explorer → Load data (Excel)
3. Build → Node → Add Nodes from Data
4. Link → Structural learning → Algorithm (PC)
5. Distributions → Parameter Learning

---

## Notación de redes bayesianas

| Símbolo | Significado |
|---------|-------------|
| Cuadrado sólido | Nodo constante (valor fijo) |
| Círculo sólido | Nodo estocástico (variable aleatoria) |
| Círculo punteado | Nodo determinista (función de otras variables) |
| Círculo sombreado | Nodo observado |
| Rectángulo discontinuo (placa) | Replicación sobre un conjunto |

---

## KL Divergence y MLE

- **MLE** (Maximum Likelihood Estimation): maximizar log-likelihood
- **Divergencia KL**: medir diferencia entre distribuciones

$$D_{KL}(P \| Q) = \sum_x P(x) \log \frac{P(x)}{Q(x)}$$

Relación clave: **minimizar KL ↔ maximizar likelihood** cuando Q es el modelo y P son los datos.

---

## Libros de referencia mencionados

- Probabilistic Graphical Models in Finance — Alexander Denev ([resumen](https://gmarti.gitlab.io/qfin/2020/08/30/book-probabilistic-graphical-models-finance-denev.html))
- Ejercicios "a mano" — Michael U. Gutmann ([arxiv](https://arxiv.org/pdf/2206.13446.pdf))
- Inferring causation from time series — J. Runge et al. 2019
