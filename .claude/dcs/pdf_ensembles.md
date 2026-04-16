# PDF: Machine Learning Ensembles

**Fuente:** `docs/raw/Ensembles.pdf`  
**Autores:** J. Muñoz y V. Laparra — Universitat de València  
**Páginas:** 58  
**Relevancia para la tarea:** Baja-media (contexto general del curso, no directamente la tarea)

---

## Estructura

1. Combining labels (votación)
2. Combining outputs (agregación continua)
3. Bagging
4. Random Forests
5. Boosting (AdaBoost, Gradient Boosting)

---

## Combinación de etiquetas (clasificación)

| Método | Descripción |
|--------|-------------|
| Majority voting | Clase con más votos |
| Weighted majority voting | Votos ponderados por calidad del clasificador |
| BKS (Behaviour Knowledge Space) | Tabla lookup: combinación de predicciones → clase más frecuente en train |

---

## Combinación de outputs (regresión / probabilidades)

| Método | Fórmula | Cuándo usar |
|--------|---------|-------------|
| Mean rule | µ = (1/T) Σ d_t | Caso por defecto (no-free-lunch) |
| Weighted average | µ = Σ w_t · d_t | Si se conoce calidad de cada predictor |
| Product rule | µ = Π d_t | Si outputs son buenas estimaciones de P(clase) |
| Trimmed mean | Ignora extremos | Si hay outliers en predicciones |
| Median | Orden intermedio | Robustez |
| Generalized mean | µ(α) — contiene todos los anteriores | α→-∞: min, α→∞: max, α=1: mean |

**Regla práctica del profesor:** usar la media por defecto; si los outputs son buenas estimaciones de posteriors, usar el producto.

---

## Bagging

- **Inventor:** Leo Breiman, 1994
- **Bagging = bootstrap + aggregation**
- Muestrear con repetición del training set (bootstrap)
- Entrenar T predictores débiles
- Agregar: media (regresión) / votación (clasificación)

**Por qué funciona:** E[φ²(x)] ≥ (E[φ(x)])² → el predictor agregado tiene menor MSE que el individual. La mejora es mayor cuando los predictores son más diversos entre sí.

**Mejores candidatos para predictores débiles:** árboles de decisión, Naive Bayes, shallow NNs.

**sklearn:**
```python
from sklearn.ensemble import BaggingClassifier, BaggingRegressor
# Modos: bootstrap (default), pasting, random subspaces, random patches
```

---

## Random Forests

- Bagging + aleatorización de features en cada split (`max_features`)
- Siempre usa árboles (de ahí el nombre)
- El subconjunto de features reduce correlación entre árboles
- **OOB (Out-of-Bag):** ~1/3 de muestras no usadas en cada árbol → estimación gratuita del error de generalización

**Ventajas vs bagging:**
- Más rápido
- Da estimación OOB del error sin necesitar test set separado
- Feature importance integrada

---

## Stacked Generalization (Wolpert, 1992)

```
Nivel 1: ensemble de T clasificadores
Nivel 2: meta-clasificador entrenado sobre las salidas de nivel 1
```

**Entrenamiento correcto (con KFold):**
1. Dividir train en K bloques
2. Para cada bloque: entrenar nivel 1 con los K-1 restantes, predecir en el bloque
3. Usar estas predicciones OOF como features para entrenar el nivel 2
4. Re-entrenar nivel 1 con todo el train

**Ejercicio del profesor:** 3-4 clasificadores de nivel 1 + LogisticRegression como nivel 2.

---

## Mixture of Experts

- Similar a stacking pero el nivel 2 (gating network) asigna pesos **por muestra**
- Los pesos son dinámicos, no fijos
- Se entrena con backpropagation o EM
