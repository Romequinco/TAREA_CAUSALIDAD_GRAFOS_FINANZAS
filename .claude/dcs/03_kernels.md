# Medidas de Dependencia con Kernels

**Referencia:** https://jejjohnson.github.io/hsic_alignment/

---

## ¿Por qué kernels?

Los kernels permiten medir dependencia en espacios de alta dimensión o con estructuras complejas sin necesidad de estimar densidades de probabilidad explícitamente. Son una alternativa poderosa a la MI cuando la estimación de densidades es difícil.

---

## HSIC — Hilbert-Schmidt Independence Criterion

**Tipo:** kernel centrado, **no normalizado**

$$\text{HSIC}(X, Y) = \frac{1}{(n-1)^2} \text{tr}(KHLH)$$

Donde:
- `K`, `L` = matrices de kernel para X e Y respectivamente
- `H` = matriz de centrado: `H = I - (1/n) 1 1ᵀ`

**Propiedades:**
- HSIC = 0 ↔ X e Y son independientes (bajo condiciones suaves)
- Escala depende del tamaño muestral → no comparar directamente entre datasets
- No normalizado: útil para tests de independencia

---

## KA — Kernel Alignment

**Tipo:** kernel **no** centrado, normalizado

$$KA(K, L) = \frac{\langle K, L \rangle_F}{\|K\|_F \|L\|_F}$$

- Producto interno de Frobenius entre matrices de kernel
- Rango: `[0, 1]`
- Mide similitud entre las matrices de kernel de X e Y

---

## CKA — Centered Kernel Alignment

**Tipo:** kernel centrado, normalizado ← **el más usado en práctica**

$$CKA(K, L) = \frac{HSIC(X, Y)}{\sqrt{HSIC(X, X) \cdot HSIC(Y, Y)}}$$

**Propiedades:**
- Rango: `[0, 1]`
- Invariante a transformaciones ortogonales e isométricas
- Permite comparar representaciones de diferentes modelos o datasets
- Ampliamente usado en deep learning para comparar capas de redes neuronales

---

## Comparación de las tres medidas

| Medida | Centrado | Normalizado | Rango | Uso típico |
|--------|----------|-------------|-------|------------|
| HSIC | Sí | No | [0, ∞) | Test de independencia |
| KA | No | Sí | [0, 1] | Comparar kernels |
| CKA | Sí | Sí | [0, 1] | Comparar representaciones |

---

## Kernels comunes

- **RBF/Gaussiano:** `K(x,y) = exp(-‖x-y‖²/2σ²)` — el más frecuente
- **Lineal:** `K(x,y) = xᵀy` — equivale a Pearson cuando se normaliza
- **Polinomial:** `K(x,y) = (xᵀy + c)^d`

---

## Contexto para el proyecto

- HSIC/CKA son útiles cuando los activos financieros tienen relaciones no lineales complejas.
- CKA es preferible a MI cuando la dimensión es alta (ej. vectores de features de múltiples activos).
- En grafos, las medidas kernel pueden usarse para definir pesos de aristas o para detectar comunidades.
