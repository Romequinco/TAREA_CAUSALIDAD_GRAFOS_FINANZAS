# Dependencia Condicional — Quitar Conexiones en el Grafo

## ¿Para qué sirve?

En un grafo probabilístico, no toda correlación observada entre X e Y es una **relación directa**. Puede ser que ambas estén correlacionadas solo porque comparten una causa común Z (variable de confusión).

**Objetivo:** decidir si una arista entre X e Y debe existir en el grafo, condicionando en las variables vecinas.

---

## Independencia Condicional

X e Y son condicionalmente independientes dado Z si:

$$P(X, Y | Z) = P(X|Z) \cdot P(Y|Z)$$

Notación: `X ⊥ Y | Z`

Si esta condición se cumple, la arista X–Y puede **eliminarse** del grafo.

---

## Tests para detectar independencia condicional

### 1. Test Chi-cuadrado (χ²)
- Para variables **categóricas**
- Contrasta si la distribución conjunta es producto de las marginales
- Referencia: https://online.stat.psu.edu/stat504/book/export/html/730

### 2. Correlación Parcial
- Para variables **continuas con relaciones lineales**
- Correlación entre X e Y eliminando el efecto lineal de Z

$$\rho_{XY|Z} = \frac{\rho_{XY} - \rho_{XZ}\rho_{YZ}}{\sqrt{(1-\rho_{XZ}^2)(1-\rho_{YZ}^2)}}$$

- Si `ρ_{XY|Z} ≈ 0` → X e Y son condicionalmente independientes dado Z
- Referencia: https://en.wikipedia.org/wiki/Partial_correlation

### 3. Información Mutua Condicional (CMI)
- Para variables con **dependencias no lineales**
- Mide la dependencia residual entre X e Y dado Z

$$I(X; Y | Z) = H(X|Z) + H(Y|Z) - H(X,Y|Z)$$

- Si `I(X;Y|Z) ≈ 0` → independencia condicional
- Referencia: https://en.wikipedia.org/wiki/Conditional_mutual_information

---

## Aplicación en algoritmos de grafos

| Algoritmo | Test usado | Tipo de grafo |
|-----------|------------|----------------|
| PC Algorithm | Correlación parcial / CMI | DAG (causal) |
| Graphical Lasso | Precisión (inversa covarianza) | Grafo no dirigido |
| MMHC | CMI | DAG |

---

## Ejemplo en el documento

El documento muestra cómo, al condicionar en variables intermedias, se puede:
- **Quitar aristas** que parecían existir (correlación espuria)
- **Mantener solo dependencias directas**

En series temporales financieras: condicionar en el mercado general puede revelar si dos activos tienen dependencia propia o simplemente siguen al mercado.

---

## Regla práctica

1. Calcular dependencia marginal entre todos los pares (X, Y)
2. Para cada par con alta dependencia, condicionar en candidatos Z
3. Si la dependencia cae a ≈ 0 al condicionar → eliminar arista X–Y
4. Repetir iterativamente (base del algoritmo PC)
