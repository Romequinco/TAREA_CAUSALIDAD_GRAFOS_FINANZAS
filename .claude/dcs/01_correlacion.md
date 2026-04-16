# Medidas de Correlación Clásica

## Correlación de Pearson

**Mide:** relación **lineal** entre dos variables X e Y.

- Rango: `[-1, 1]`
- `r = 1` → perfectamente correlacionadas positivamente
- `r = 0` → sin correlación lineal (¡no implica independencia!)
- `r = -1` → perfectamente correlacionadas negativamente

**Fórmula:**

$$r = \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum(x_i-\bar{x})^2 \sum(y_i-\bar{y})^2}}$$

**Limitación clave:** Solo detecta relaciones lineales. Dos variables con dependencia no lineal fuerte pueden tener Pearson ≈ 0.

> Ejemplo del documento: Pearson = 0.30 pero Spearman = 0.76 en el mismo dataset → hay una relación de orden/monotónica que Pearson pierde.

---

## Correlación de Spearman

**Mide:** relación **monotónica** (de orden) entre variables.

- Equivale a aplicar Pearson sobre los **rangos** de los valores, no los valores en sí.
- Más robusta a outliers que Pearson.
- Rango: `[-1, 1]`

**Cuándo usarlo:**
- Datos con outliers severos
- Relaciones monotónicas no lineales (ej. precios de activos en horizontes largos)
- Datos ordinales

---

## Comparación

| Medida | Tipo de relación | Robustez a outliers | Detecta no-linealidad |
|--------|-----------------|---------------------|-----------------------|
| Pearson | Lineal | Baja | No |
| Spearman | Monotónica | Alta | Parcial |
| Información Mutua | Cualquiera | Media | Sí |
| Kernel (HSIC/CKA) | Cualquiera | Depende del kernel | Sí |

---

## Contexto para el proyecto (finanzas/grafos)

- Pearson es la base de muchas matrices de correlación financiera (ej. portafolios Markowitz).
- Spearman es preferible en retornos de activos con colas pesadas (fat tails).
- Ambas son **simétricas**: no dicen nada sobre dirección causal.
- En grafos no dirigidos, una correlación alta puede justificar una arista; pero **no determina la dirección**.
