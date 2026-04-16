# PDF: XAI — Explicabilidad en ML

**Fuentes:**
- `docs/raw/XAI_surrogates_sokol.pdf` — Kacper Sokol & Peter Flach
- `docs/raw/MIAX_intro.pdf` — Raul Santos-Rodriguez, University of Bristol

**Relevancia para la tarea:** Baja (contexto del módulo, no directamente en B3-T3)

---

## Tipos de explicabilidad (Sokol & Flach)

| Tipo | Descripción |
|------|-------------|
| Transparencia | Insight de complejidad arbitraria sobre el sistema |
| Simulatabilidad | Modelo tan simple que el humano puede simularlo mentalmente |
| Surrogate | Modelo proxy local que aproxima el modelo caja negra |
| Counterfactual | "¿Qué habría cambiado para obtener otro resultado?" |

---

## Surrogate Explainers — los 4 principales

| Método | Paper | Tipo |
|--------|-------|------|
| **LIME** | Ribeiro et al., 2016 | Local, model-agnostic |
| **Anchor** | Ribeiro et al., 2018 | Local, reglas de si/no |
| **SHAP** | Lundberg & Lee, 2017 | Local/global, Shapley values |
| **RuleFit** | Friedman & Popescu, 2008 | Global, reglas |

---

## LIME — Local Interpretable Model-agnostic Explanations

- Perturba la instancia de interés
- Entrena un modelo lineal local sobre las perturbaciones
- Los coeficientes del modelo lineal son la explicación

```python
from lime.lime_tabular import LimeTabularExplainer

explainer = LimeTabularExplainer(
    Xtr.values,
    feature_names=feature_names,
    class_names=class_names,
    mode='classification'  # o 'regression'
)
exp = explainer.explain_instance(x, model.predict_proba, num_features=10)
```

---

## SHAP — SHapley Additive exPlanations

- Basado en Shapley values de teoría de juegos
- **Kernel SHAP:** model-agnostic, más lento pero teóricamente fundamentado
- Contribución marginal de cada feature promediada sobre todas las combinaciones posibles

```python
import shap

background = shap.sample(Xtr, 100)  # dataset de referencia reducido
explainer = shap.Explainer(model.predict_proba, background)
shap_values = explainer(Xts.iloc[:10])

# Visualizaciones
shap.plots.waterfall(shap_values[0, :, 1])  # local
shap.plots.beeswarm(shap_values)             # global
```

---

## Weak Supervision (MIAX — Santos-Rodriguez)

Contexto: cuando las etiquetas del training set son **imperfectas o parciales**.

| Escenario | Descripción |
|-----------|-------------|
| Noisy labels | Etiquetas con ruido aleatorio o sistemático |
| Partial labels | Solo se sabe que la clase está en un subconjunto |
| Semi-supervised | Mezcla de etiquetas verdaderas y débiles |
| Label proportions | Solo proporciones de clase por grupo, no por instancia |

**Mixing matrix M:** M_ij = P(weak_label=b_i | true_label=y_j) — modela la relación entre etiqueta débil y verdadera.

**Teorema clave (Cid-Sueiro et al., ECML 2015):**
> Una función de pérdida débil Ψ es M-propia si y solo si la función de pérdida equivalente Ψ̃ = Mᵀ Ψ es propia (estándar).

---

## Counterfactuals — FACE

- "¿Por qué no se aprobó mi hipoteca?" → qué cambios mínimos llevarían a aprobación
- Problema: los counterfactuals simples pueden estar fuera de la distribución de datos
- **FACE** (Feasible and Actionable Counterfactual Explanations): usa distancias ponderadas por densidad para encontrar caminos factibles

Referencia: Poyiadzi et al., AAAI/AIES 2020

---

## Crítica a los explainers post-hoc (Sokol)

> "Post-hoc explainers have poor fidelity."

- Los surrogates son aproximaciones locales — no garantizan fidelidad global
- Un modelo inherentemente interpretable (árbol, regresión lineal) es mejor que un surrogate sobre un modelo opaco
- La explicabilidad requiere esfuerzo de diseño, no es automática
