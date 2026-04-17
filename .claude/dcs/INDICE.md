# Índice de Contexto — .claude/dcs/

Carpeta de contexto para Claude. Organizada por secciones.  
Actualizado: 2026-04-17

---

## SECCIÓN A — LA TAREA

| Archivo | Descripción |
|---------|-------------|
| [contexto_tarea.md](contexto_tarea.md) | Enunciado completo del Taller B3-T3. Entrega **23 Abril**. Qué hay que entregar, criterios, estructura del notebook |
| [contexto_taller_causalidad.md](contexto_taller_causalidad.md) | **Transcripción de Valero resolviendo el taller en vivo.** Flujo de código real, variaciones demostradas, bugs detectados, explicación del Factor Mirage, datos usados, filosofía de evaluación |

---

## SECCIÓN B — TEORÍA CORE (PDFs del profesor)

| Archivo | Fuente PDF | Contenido |
|---------|-----------|-----------|
| [01_correlacion.md](01_correlacion.md) | Medidas_de_dependencia_2025.pdf | Pearson vs Spearman, cuándo usar cada uno |
| [02_teoria_informacion.md](02_teoria_informacion.md) | Medidas_de_dependencia_2025.pdf | Shannon, Entropía, MI, ICC, MI condicional |
| [03_kernels.md](03_kernels.md) | Medidas_de_dependencia_2025.pdf | HSIC, KA, CKA — medidas no paramétricas |
| [04_dependencia_condicional.md](04_dependencia_condicional.md) | Medidas_de_dependencia_2025.pdf | Chi-square, correlación parcial, MI condicional para grafos |
| [05_tests_hipotesis.md](05_tests_hipotesis.md) | Medidas_de_dependencia_2025.pdf | H0/Ha, p-valor, errores tipo I/II, Bonferroni/FDR |
| [06_causalidad.md](06_causalidad.md) | Medidas_de_dependencia_2025.pdf | Granger, Directed Info, Additive Noise Models |
| [00_resumen_general.md](00_resumen_general.md) | Medidas_de_dependencia_2025.pdf | Índice y estructura del PDF de medidas |
| [pdf_modelos_grafos_probabilisticos.md](pdf_modelos_grafos_probabilisticos.md) | Modelos_de_Gafos_Probabilisticos_2025.pdf | Redes Bayesianas, algoritmo PC, pgmpy, notación |

---

## SECCIÓN C — BIBLIOGRAFÍA PRINCIPAL DE LA TAREA

| Archivo | Fuente PDF | Contenido |
|---------|-----------|-----------|
| [resumen_lopez_de_prado.md](resumen_lopez_de_prado.md) | rf_lopezdeprado_causalityprimer_online.pdf | Factor Mirage, collider bias, ejemplo Barra, best practices |

---

## SECCIÓN D — CÓDIGO DEL PROFESOR (Notebooks)

| Archivo | Notebooks fuente | Contenido |
|---------|-----------------|-----------|
| [nb_pgmpy_grafos_causales.md](nb_pgmpy_grafos_causales.md) | VALERO_pgmpy_Structure Learning_*.ipynb | Cómo usa el profesor PC y HillClimb, preprocesamiento, visualización |
| [nb_estilo_codigo_profesor.md](nb_estilo_codigo_profesor.md) | Todos los notebooks | Patrones de código: estructura, bagging, RF, stacking |
| [nb_librerias_instalacion.md](nb_librerias_instalacion.md) | Todos los notebooks | Librerías, API pgmpy, tests CI, carga de datos financieros |

---

## SECCIÓN E — CONTEXTO DEL CURSO (PDFs secundarios)

| Archivo | Fuente PDF | Contenido |
|---------|-----------|-----------|
| [pdf_ensembles.md](pdf_ensembles.md) | Ensembles.pdf | Bagging, RF, Stacking, combinación de outputs |
| [pdf_xai_miax.md](pdf_xai_miax.md) | XAI_surrogates_sokol.pdf + MIAX_intro.pdf | LIME, SHAP, counterfactuals, weak supervision |

---

## SECCIÓN F — BUENAS PRÁCTICAS Y REFERENCIA RÁPIDA

| Archivo | Contenido |
|---------|-----------|
| [07_buenas_practicas.md](07_buenas_practicas.md) | Pipeline recomendado, errores frecuentes, tabla de qué medida usar cuándo, todas las referencias |

---

## Prioridad de consulta para la tarea B3-T3

```
1. contexto_taller_causalidad.md  → FUENTE PRIMARIA: Valero resolviendo el taller en vivo
2. contexto_tarea.md              → enunciado formal y criterios de evaluación
3. resumen_lopez_de_prado.md      → bibliografía base, concepto Factor Mirage
4. nb_pgmpy_grafos_causales.md    → API pgmpy detallada + flujo financiero
5. 06_causalidad.md               → teoría de Granger y DAGs
6. 04_dependencia_condicional.md  → cómo quitar/añadir aristas
7. 07_buenas_practicas.md         → errores a evitar
```
