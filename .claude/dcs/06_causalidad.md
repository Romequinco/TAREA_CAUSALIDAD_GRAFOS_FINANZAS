# Causalidad — Poner Flechas en el Grafo

## Principio fundamental

> **Correlación (o dependencia) NO implica causalidad.**

Dos variables pueden estar correlacionadas porque:
1. X causa Y
2. Y causa X
3. Ambas tienen una causa común Z (confusión)
4. Es una correlación espuria (coincidencia en la muestra)

El objetivo de las medidas direccionales es distinguir el caso 1 del 2.

Referencia general: https://en.wikipedia.org/wiki/Exploratory_causal_analysis

---

## 1. Causalidad de Granger

**Idea:** X "causa Granger" a Y si el pasado de X ayuda a predecir el futuro de Y, más allá de lo que ya predice el propio pasado de Y.

**Procedimiento:**

1. Modelo restringido: regresión de Y_t sobre sus propios rezagos → residuos RSS₁
2. Modelo completo: regresión de Y_t sobre rezagos de Y **y** X → residuos RSS₂
3. Test F: ¿mejora significativamente al añadir X?

$$F = \frac{(RSS_1 - RSS_2)/p}{RSS_2/(n-2p-1)}$$

Si p-valor < α → X causa Granger a Y.

**Importante:** Es causalidad en sentido predictivo, **no** causalidad física o mecanística.

**Ejemplo del documento:** Los precios del petróleo causan Granger a los precios del oro (al 10%), pero no al revés.

Referencia: https://en.wikipedia.org/wiki/Granger_causality  
Guía práctica: https://www.aptech.com/blog/introduction-to-granger-causality/

---

## 2. Directed Information (Información Dirigida)

$$I(X^n \to Y^n) = \sum_{i=1}^n I(X^i; Y_i | Y^{i-1})$$

- Mide el flujo de información de X hacia Y en el tiempo
- A diferencia de la MI clásica (simétrica), la información dirigida es **asimétrica**
- Permite detectar si hay más información fluyendo de X→Y que de Y→X

Referencia: https://en.wikipedia.org/wiki/Directed_information  
Paper: "Universal Estimation of Directed Information" (Jiao et al.) — https://arxiv.org/pdf/1201.2334.pdf

---

## 3. Interaction Information (Información de Interacción)

$$II(X; Y; Z) = I(X; Y | Z) - I(X; Y)$$

- **Puede ser negativa** (a diferencia de MI)
- Negativa → Z **media** la relación entre X e Y (chain)
- Positiva → Z crea **sinergia** entre X e Y (collider)
- Útil para identificar estructuras causales de tres variables

Referencia: https://en.wikipedia.org/wiki/Interaction_information

---

## 4. Causalidad desde los Residuos (Additive Noise Models)

**Idea:** Si X → Y, entonces el modelo `y = f(x) + n` tendrá residuos independientes de X. El modelo inverso `x = f(y) + n` **no** tendrá residuos independientes de Y.

**Procedimiento:**
1. Ajustar `y = f(x) + n` y calcular residuos `n̂ = y - f̂(x)`
2. Testear independencia entre `n̂` y `x`
3. Ajustar `x = f(y) + n` y repetir
4. La dirección donde los residuos son independientes de la entrada es la **causal**

**Referencia clave:**  
Mooij et al. "Distinguishing cause from effect using observational data: methods and benchmarks" — https://arxiv.org/pdf/1412.3773v1.pdf

---

## Resumen: ¿Qué método usar?

| Método | Requisito | Para qué sirve |
|--------|-----------|----------------|
| Granger causality | Series temporales | Causalidad predictiva X→Y en tiempo |
| Directed Information | Series temporales | Flujo de información asimétrico |
| Interaction Information | Tres variables | Detectar mediadores vs colisores |
| Additive Noise Models | Datos observacionales | Dirección causal X→Y vs Y→X |

---

## Contexto para el proyecto (finanzas)

- En mercados financieros, Granger causality entre activos puede revelar dependencias de liderazgo (lead-lag).
- Los ANM pueden usarse para distinguir si el mercado lidera a un sector o viceversa.
- La información dirigida es más robusta que Granger para relaciones no lineales.
- Todos estos métodos generan las **flechas** de un DAG (Directed Acyclic Graph), el siguiente paso tras construir el grafo no dirigido.
