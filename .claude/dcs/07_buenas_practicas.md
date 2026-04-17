# Buenas Prácticas y Errores Comunes

## Flujo recomendado para analizar dependencia

```
Variables → Dependencia marginal → Dependencia condicional → Causalidad
              (Pearson/MI/HSIC)     (MI condicional/parcial)   (Granger/ANM)
                    ↓                        ↓                      ↓
              Aristas del grafo      Filtrar aristas         Orientar flechas
```

---

## Selección de medida según el contexto

| Situación | Medida recomendada |
|-----------|-------------------|
| Relación claramente lineal | Pearson |
| Relación monotónica, outliers | Spearman |
| Relación no lineal desconocida | Información Mutua o CKA |
| Alta dimensionalidad | CKA / HSIC |
| Series temporales, causalidad | Granger + Directed Information |
| Datos observacionales, dir. causal | Additive Noise Models |
| Tests en grafos con muchas variables | MI condicional + corrección FDR |

---

## Errores frecuentes a evitar

### 1. Confundir correlación con causalidad
Pearson = 0.9 entre A y B no significa que A cause B. Siempre investigar si hay una variable de confusión.

### 2. Mal interpretar el p-valor
`p < 0.05` NO significa "hay 95% de probabilidad de dependencia real".  
Significa: "este resultado sería raro si no hubiera dependencia".

### 3. Usar solo Pearson en datos financieros
Los retornos financieros tienen colas pesadas y relaciones no lineales (especialmente en crisis). Pearson subestima la dependencia en los extremos. Usar Spearman o MI.

### 4. Ignorar la corrección por múltiples tests
Con 50 activos hay 1225 pares. Al nivel α = 0.05 se esperan ~61 falsos positivos por azar.  
**Solución:** Bonferroni (`α' = α/m`) o FDR de Benjamini-Hochberg.

### 5. Confundir Granger causality con causalidad real
Granger mide **predictibilidad temporal**. Puede ser que X prediga a Y porque ambos son causados por Z con distinto retraso.

### 6. No verificar estacionariedad en series temporales
Granger causality asume series estacionarias. Hacer test ADF/KPSS antes.

### 7. Escala del HSIC
HSIC no normalizado crece con el tamaño muestral. Siempre usar CKA (versión normalizada) para comparar.

---

## Bugs y errores detectados en vivo (clase Valero, 17 abril 2026)

- **Sobreescribir el PNG**: si llamas varias veces a `model.to_graphviz().draw("model.png")` con el mismo nombre de archivo, el visualizador puede seguir mostrando el anterior. Usar nombres distintos: `"model_pc.png"`, `"model_hc.png"`, etc.
- **Caracteres especiales en columnas**: pgmpy falla con guiones (`BRK-B`) y circunflejos (`^GSPC`). Siempre limpiar con `c.replace('-', '').replace('^', '')` antes de pasar a pgmpy.
- **Trabajar con precios en vez de retornos**: la causalidad y correlación en finanzas se mide sobre retornos (`pct_change()`), nunca sobre precios en niveles (no estacionarios).
- **Lag con NaN al final**: `df.shift(-1)` genera un NaN en la última fila. Aplicar `dropna()` antes de pasar a pgmpy.
- **Robustez ante estocasticidad (técnica López de Prado)**: el algoritmo puede dar grafos distintos en cada ejecución. Solución: ejecutar N=50 veces y quedarse solo con las aristas que aparecen en ≥ 2/3 de las iteraciones (~35 de 50).

---

## Pipeline típico para grafos causales en finanzas

```python
# 1. Calcular matriz de dependencia (ej. Spearman)
dep_matrix = spearman_matrix(returns)

# 2. Test de significancia + corrección FDR
sig_matrix = fdr_correction(dep_matrix, alpha=0.05)

# 3. Construir grafo no dirigido con aristas significativas
G = build_graph(sig_matrix)

# 4. Aplicar tests de independencia condicional para filtrar aristas
G = pc_algorithm(G, test="partial_correlation")

# 5. Orientar flechas con Granger o ANM
G_directed = orient_edges(G, method="granger", max_lag=5)
```

---

## Referencias clave del documento

| Tema | Referencia |
|------|------------|
| Información Mutua en Earth Data | Johnson et al. (2021) IEEE GRSL — https://arxiv.org/pdf/2010.06476.pdf |
| HSIC y variantes | https://jejjohnson.github.io/hsic_alignment/ |
| Information Correlation Coefficient | Linfoot (1957) — https://encyclopediaofmath.org/wiki/Information_correlation_coefficient |
| Directed Information | https://arxiv.org/pdf/1201.2334.pdf |
| Granger causality | https://www.aptech.com/blog/introduction-to-granger-causality/ |
| Additive Noise Models | Mooij et al. — https://arxiv.org/pdf/1412.3773v1.pdf |
| Correlación parcial | https://en.wikipedia.org/wiki/Partial_correlation |
| MI condicional | https://en.wikipedia.org/wiki/Conditional_mutual_information |
