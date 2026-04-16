# Teoría de la Información aplicada a Dependencia

## Concepto base: Información de Shannon (1948)

> "A Mathematical Theory of Communication" — Claude E. Shannon

**Principio:** La información de un evento es proporcional a la inversa de su probabilidad.

$$I(x) = \log\left(\frac{1}{p(x)}\right) = -\log(p(x))$$

- Evento muy probable → poca información (ya lo esperábamos)
- Evento improbable → mucha información (nos sorprende)
- Unidad: **bits** (si log en base 2)

---

## Entropía H(X)

**Valor promedio de información** de una variable aleatoria X.

$$H(X) = -\sum_x p(x) \log p(x)$$

- Mide la **incertidumbre** de una variable.
- H = 0 → la variable es determinista (sin sorpresa)
- H máxima → distribución uniforme (máxima incertidumbre)
- Se mide en **bits**; útil para compresión y almacenamiento de datos.

---

## Información Mutua I(X; Y)

**Mide cuánta información comparten dos variables.**

$$I(X, Y) = \sum_{x,y} p(x,y) \log \frac{p(x,y)}{p(x)p(y)}$$

- `I(X,Y) = 0` ↔ X e Y son **independientes**
- `I(X,Y) > 0` → hay dependencia (de cualquier tipo, lineal o no)
- **Simétrica:** I(X,Y) = I(Y,X)
- **No negativa** siempre
- Relación con entropía: `I(X,Y) = H(X) + H(Y) - H(X,Y)`

**Ventaja sobre Pearson:** detecta **cualquier tipo de dependencia**, no solo lineal.

---

## ICC — Information Correlation Coefficient

Convierte la Información Mutua a un coeficiente interpretable como correlación en `[-1, 1]`:

$$R(X, Y) = \sqrt{1 - e^{-2 I(X,Y)}}$$

- Propuesto por E.H. Linfoot (1957)
- Rango: `[0, 1]` (la raíz elimina el signo)
- ICC = 0 → independencia; ICC = 1 → dependencia perfecta
- Permite comparar la MI entre pares de variables en la misma escala

---

## Información Mutua Condicional I(X; Y | Z)

$$I(X; Y | Z) = H(X|Z) + H(Y|Z) - H(X,Y|Z)$$

- Mide la dependencia entre X e Y **dado que conocemos Z**
- Fundamental para **quitar aristas en grafos**: si `I(X;Y|Z) ≈ 0`, la conexión X-Y puede eliminarse (Z la explica).
- Base del algoritmo PC y otros métodos de aprendizaje de grafos causales.

---

## Contexto para el proyecto

- La MI es más apropiada que Pearson para relaciones no lineales en mercados financieros.
- El ICC permite ordenar pares de activos por grado de dependencia.
- La MI condicional es el ingrediente clave para determinar **dependencias directas vs. indirectas** en grafos probabilísticos.
