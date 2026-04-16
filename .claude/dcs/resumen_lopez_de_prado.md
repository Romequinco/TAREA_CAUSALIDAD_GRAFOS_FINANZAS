# Resumen: "Causality and Factor Investing: A Primer"
### López de Prado & Zoonekynd — CFA Institute Research Foundation, 2025

---

## Idea central del paper

> **La econometría clásica del factor investing ignora la causalidad. Esto genera modelos que funcionan en backtest pero fallan en producción.**

El paper argumenta que el problema raíz del bajo rendimiento del factor investing no es solo el p-hacking o el overfitting, sino algo más profundo: **confundir asociación con causalidad**.

---

## 1. El problema: del "Factor Zoo" al "Factor Mirage"

### Factor Zoo (Cochrane, 2011)
Cientos de anomalías publicadas compiten por atención. La mayoría no sobrevive la replicación ni la implementación real.

### `> RELEVANTE PARA NOSOTROS` — Factor Mirage (concepto clave del paper)
Un **factor mirage** es un hallazgo empírico que:
- Parece estadísticamente robusto por estándares convencionales
- Pero es **estructuralmente inválido** porque malrepresenta las relaciones causales
- Funciona bien en backtest y cross-validation
- **Falla en live trading** porque las asociaciones no causales no se pueden monetizar

**Origen de los mirages — dos errores de especificación:**

| Error | Definición | Consecuencia |
|-------|-----------|--------------|
| **Confounder bias** | No controlar por una variable que causa TANTO al factor como a los retornos | Coeficiente sesgado en magnitud y posiblemente en signo |
| **Collider bias** | Controlar por una variable que es CONSECUENCIA tanto del factor como de los retornos | Introduce correlación no causal, infla R², baja p-values — **puede cambiar el signo del coeficiente** |

---

## 2. Por qué falla la econometría clásica

Los métodos estándar (OLS, stepwise regression, significance testing) asumen especificación correcta del modelo. En finanzas, esto casi nunca se verifica causalmente.

### El two-pass regression (estándar en asset pricing)
1. Regresión temporal: activos vs. factores → estima betas
2. Regresión cross-seccional: retornos vs. betas → estima primas de riesgo

**El problema:** la selección de controles se basa en maximizar R² (criterio asociacional), NO en lógica causal.

### Métricas estándar que premian la malespecificación
- R² ajustado
- AIC / BIC
- t-statistics

Todas estas métricas **recompensan incluir colliders** (sube el R²) aunque eso destruya la validez causal del modelo.

### `> RELEVANTE — Granger causality NO es causalidad real`
Granger (1969) define causalidad como "poder predictivo de X sobre Y controlando por lags de Y". Esto **no es causalidad counterfactual** (Pearl, 2009). Un gallo no hace salir el sol aunque lo "cause" en sentido Granger.

---

## 3. `> RELEVANTE PARA NOSOTROS` — Ejemplo con Barra Factors (PC Algorithm)

Este es el ejemplo central del paper y el más cercano a nuestra tarea.

### Qué hicieron
Aplicaron el **algoritmo PC (Peter-Clark)** a las series temporales de retornos diarios de los factores de riesgo de **85 modelos Barra**. El resultado es un **DAG (Grafo Acíclico Dirigido)** que muestra la estructura causal entre factores.

### Factores Barra analizados
```
SIZE, MIDCAP, BETA, BTOP (book-to-price), EARNYILD, EARNQLTY,
EARNVAR, GROWTH, INVSQLTY, LEVERAGE, LIQUIDTY, LTREVRSL,
MOMENTUM, PROFIT, RESVOL, DIVYILD
```

### Resultado (Exhibit 1)
El grafo agrega los edges presentes en al menos 1/3 de los modelos. Ejemplo de estructura descubierta:
- SIZE → RESVOL, LIQUIDTY, MIDCAP, BETA
- MOMENTUM → GROWTH, EARNYILD
- LEVERAGE → GROWTH, EARNYILD
- BTOP → EARNQLTY, DIVYILD, EARNYILD

### Matriz de adyacencia (Exhibit 2)
La matriz direccional es **asimétrica** — eso es exactamente lo que pedimos construir en la tarea. Las entradas bajo la diagonal indican edges donde el algoritmo no pudo determinar dirección: (SIZE, LIQUIDITY) y (SIZE, RESVOL).

### Implicación práctica clave
> Para invertir en un factor, hay que **controlar por sus ancestros (causas)**, NO por sus descendientes (efectos = colliders).

Ejemplo: para modelar GROWTH correctamente:
- ✅ Controlar por: MOMENTUM, LEVERAGE, LTREVRSL, PROFIT, INVSQLTY
- ❌ NO controlar por: EARNYILD, BTOP, EARNQLTY, DIVYILD (son descendientes → colliders)

### Exhibit 4 — El modelo mal especificado tiene MAYOR R²
| Modelo | R² ajustado |
|--------|------------|
| Con colliders (mal especificado) | 8.5% |
| Sin colliders (correcto) | 7.8% |

**Conclusión:** Un R² más alto no significa mejor modelo. Puede significar collider bias.

### `> RELEVANTE — Exhibit 5 — Factor Mirage en acción`
26 casos en Barra donde **añadir un collider cambia el signo del coeficiente estimado**:

Ejemplos destacados:
- **LIQUIDTY**: con RESVOL → beta = +0.08; añadiendo collider BETA → beta = **-0.04** (¡signo contrario!)
- **EARNYILD**: con GROWTH → beta = +0.08; añadiendo collider EARNQLTY → beta = **-0.03**
- **GROWTH**: con MOMENTUM → beta = +0.009; añadiendo collider BTOP → beta = **-0.0006**

Esto significa que un gestor podría estar **comprando lo que debería vender** por culpa del collider bias.

---

## 4. Coste económico de ignorar la causalidad

| Problema | Descripción |
|----------|-------------|
| **Mala asignación de capital** | Se invierte en estrategias estadísticamente significativas pero económicamente vacías |
| **Leverage oculto** | Modelos con los mismos errores de especificación acumulan exposiciones de riesgo similares sin saberlo |
| **Exceso de rotación** | Señales espurias generan operaciones innecesarias → costes de transacción |
| **Falta de persistencia** | Modelos no causales fallan cuando cambia el régimen económico |
| **Pérdida de confianza** | El backtesting consistentemente supera el live performance → los clientes dejan de creer en el factor investing |

---

## 5. `> RELEVANTE — Best Practices (checklist para nuestra tarea)`

El paper propone 7 pasos. Los más relevantes para nosotros:

### Step 2: Causal Discovery
- ¿Se construyó un grafo causal?
- ¿Qué algoritmos se usaron? → **PC, LiNGAM, GES** ← esto es exactamente lo que pedimos hacer
- ¿Qué soporte económico tiene el grafo?

### Step 3: Causal Adjustment
- ¿Qué método de ajuste se usó? → backdoor, front door, instrumental variable
- ¿Se verificó con software de do-calculus? (DAGitty, DoWhy, networkX)
- ¿Se identificaron y excluyeron colliders?

### Step 6: Backtesting
- ¿Se usó el grafo causal para simular escenarios?

### Step 7: Multiple Testing Adjustment
- ¿Se reportó el Deflated Sharpe Ratio (DSR)?

---

## 6. Conclusión del paper

> "Investment factors exist, but the way researchers build and evaluate them is flawed."

El **factor mirage** es consecuencia de:
1. **Collider bias** — controlar por variables que son consecuencias del factor
2. **Confounder bias** — no controlar por variables que confunden la relación

La solución es adoptar **causal factor modeling**: construir DAGs con algoritmos de descubrimiento causal antes de especificar cualquier modelo de regresión.

---

## Conexión directa con nuestra tarea

| Lo que pide el taller | Lo que dice el paper |
|-----------------------|----------------------|
| Matriz de correlaciones | Punto de partida — muestra asociación, NO causalidad |
| Matriz direccional | Paso intermedio — cuantifica asimetría temporal entre variables |
| DAG con algoritmo causal | **Algoritmo PC** es exactamente el que usa López de Prado en el ejemplo de Barra |
| Análisis con/sin LAG | Relacionado con Granger causality vs. causalidad counterfactual |
| Distintos parámetros/métodos | PC, LiNGAM, GES producen DAGs distintos — importante mostrarlo |
| **Factor Mirage (extra)** | **Es el concepto central del paper** — cambio de signo al añadir un collider |

### `> PARA EL VÍDEO — argumento financiero clave`
La correlación entre dos factores no implica que uno cause al otro. El DAG revela la dirección y permite identificar:
- Qué variables son causas (ancestros) → controlar por ellas es correcto
- Qué variables son efectos (descendientes/colliders) → controlarlas introduce sesgo

Esto explica por qué estrategias de factor investing que parecen robustas en backtest fallan en producción: estaban monetizando una asociación espuria creada por un collider.

---

## Algoritmos de descubrimiento causal mencionados

| Algoritmo | Nombre completo | Librería Python |
|-----------|----------------|-----------------|
| **PC** | Peter-Clark | `causallearn` |
| **LiNGAM** | Linear Non-Gaussian Acyclic Model | `lingam` |
| **GES** | Greedy Equivalence Search | `causallearn` |
| **NOTEARS** | No-Tears (gradient-based) | `castle` (gcastle) |

> El paper usa **PC** en su ejemplo con Barra → recomendable usarlo como algoritmo principal en nuestra tarea.
