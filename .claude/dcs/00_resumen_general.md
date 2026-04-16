# Medidas de Dependencia — Resumen General

**Fuente:** `docs/raw/Medidas_de_dependencia_2025.pdf`  
**Autor:** Valero Laparra  
**Páginas:** 49

---

## ¿De qué trata el documento?

El documento es una presentación/curso sobre **medidas de dependencia estadística** entre variables, con aplicaciones a series temporales, grafos probabilísticos y causalidad. Cubre desde métricas clásicas (Pearson, Spearman) hasta métodos modernos basados en teoría de la información y kernels.

---

## Estructura del documento

| Bloque | Contenido |
|--------|-----------|
| 1. Correlación clásica | Pearson (relación lineal), Spearman (relación de orden) |
| 2. Independencia estadística | Definición formal, cuándo dos variables son independientes |
| 3. Teoría de la información | Información de Shannon, Entropía, Información mutua, ICC |
| 4. Medidas con Kernels | HSIC, Kernel Alignment (KA), CKA |
| 5. Series temporales | Ejemplos prácticos: ventana temporal, comparación de modelos, redundancia |
| 6. Dependencia condicional | Quitar conexiones en grafos: chi-square, correlación parcial, MI condicional |
| 7. Tests de hipótesis | H0, Ha, p-valor, intervalos de confianza |
| 8. Causalidad | Directed information, Granger causality, Causality from residuals |

---

## Idea central

> **Dependencia ≠ Causalidad**

Medir si dos variables están relacionadas es distinto a saber cuál causa a cuál. El documento progresa desde "¿están relacionadas?" hasta "¿cuál causa a cuál?".

---

## Archivos relacionados en esta carpeta

- [01_correlacion.md](01_correlacion.md) — Pearson y Spearman
- [02_teoria_informacion.md](02_teoria_informacion.md) — Entropía, MI, ICC
- [03_kernels.md](03_kernels.md) — HSIC, KA, CKA
- [04_dependencia_condicional.md](04_dependencia_condicional.md) — Tests para grafos
- [05_tests_hipotesis.md](05_tests_hipotesis.md) — p-valor y significancia
- [06_causalidad.md](06_causalidad.md) — Granger, Directed Info, residuos
- [07_buenas_practicas.md](07_buenas_practicas.md) — Guía de uso y errores comunes
