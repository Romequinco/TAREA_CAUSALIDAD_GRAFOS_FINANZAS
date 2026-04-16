# Tests de Hipótesis en Dependencia Estadística

## Lógica general: reducción al absurdo

| Símbolo | Nombre | Descripción |
|---------|--------|-------------|
| H₀ | Hipótesis nula | Lo que **no** queremos que sea verdad (ej: "no hay dependencia") |
| Hₐ | Hipótesis alternativa | Lo que queremos demostrar (ej: "sí hay dependencia") |

El test intenta **rechazar H₀**. Si lo conseguimos, aceptamos Hₐ.

Ejemplo estándar:
- H₀: `ρ = 0` (no hay correlación)
- Hₐ: `ρ ≠ 0` (hay correlación)

---

## P-valor

> "Probabilidad de obtener un resultado tan extremo (o más) como el observado, **asumiendo que H₀ es verdadera**."

$$p\text{-valor} = P(\text{estadístico} \geq t_{\text{observado}} \mid H_0 \text{ cierta})$$

**Interpretación:**
- p-valor bajo (< 0.05 típicamente) → el resultado es muy improbable bajo H₀ → rechazamos H₀
- p-valor alto → no podemos rechazar H₀ (pero no la "aceptamos")

---

## ADVERTENCIA CRÍTICA

$$P(\text{observación} \mid H_0) \neq P(H_0 \mid \text{observación})$$

El p-valor **no** es la probabilidad de que H₀ sea verdadera. Es uno de los errores más frecuentes en ciencia de datos.

- p = 0.03 **no** significa "hay 97% de probabilidad de que exista dependencia"
- Solo dice que ese resultado es raro si H₀ fuera cierta

---

## Intervalos de confianza

Un intervalo de confianza del 95% para un parámetro θ significa:

> Si repitiéramos el experimento muchas veces, el 95% de los intervalos calculados contendrían el verdadero valor de θ.

**No** significa que haya 95% de probabilidad de que θ esté en ese intervalo específico.

---

## Errores de tipo I y II

| Error | Descripción | Consecuencia |
|-------|-------------|--------------|
| Tipo I (α) | Rechazar H₀ cuando es verdadera | Falso positivo de dependencia |
| Tipo II (β) | No rechazar H₀ cuando es falsa | Perderse una dependencia real |

- Nivel de significancia α = 0.05 controla el error tipo I
- El **poder** del test (1-β) aumenta con más datos

---

## Aplicación en grafos y dependencia

- Al hacer múltiples tests (un test por par de variables), el error tipo I se acumula → usar **corrección de Bonferroni** o FDR (False Discovery Rate).
- En grafos con n variables, hay `n(n-1)/2` pares → ajuste es esencial.
- Los tests de Granger causality también usan este framework (p-valor del test F sobre los coeficientes de regresión).

---

## Referencia del documento (ejemplo Granger)

> "Rechazar la hipótesis nula de que los precios del petróleo **no** causan Granger a los precios del oro al nivel del 10%."

Esto significa: hay evidencia estadística de que el petróleo tiene poder predictivo sobre el oro (pero no al revés).
