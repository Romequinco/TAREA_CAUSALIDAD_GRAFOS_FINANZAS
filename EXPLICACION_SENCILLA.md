# Explicación del Trabajo: Causalidad entre Variables Financieras con Grafos

---

## ¿De qué trata esto en palabras normales?

Este trabajo intenta responder una pregunta muy concreta: **¿cuando sube el VIX, eso hace que baje el SP500, o simplemente suelen moverse juntos por coincidencia?**

Es decir, no nos conformamos con saber que dos variables "van juntas". Queremos saber si una **provoca** a la otra. La herramienta que usamos para esto se llama **grafo causal** (o DAG), y es básicamente un diagrama con flechas que dice "esto influye en aquello".

---

## Las 5 variables que estudiamos

Descargamos datos de 5 activos financieros muy conocidos, desde el año 2000 hasta 2024:

| Nombre | ¿Qué es? |
|--------|----------|
| **SP500** | El índice bursátil más importante de EE.UU. (las 500 mayores empresas) |
| **VIX** | El "índice del miedo": mide cuánto nerviosismo hay en el mercado |
| **GLD** | El precio del oro |
| **TLT** | El precio de los bonos del gobierno de EE.UU. a largo plazo |
| **DXY** | La fuerza del dólar americano frente a otras monedas |

---

## Sección 0 y 1 — Preparar las herramientas

Antes de hacer nada, instalamos las librerías de Python que vamos a necesitar. Las más importantes son:

- **pgmpy**: la que sabe construir grafos causales (algoritmos PC y HillClimbSearch)
- **yfinance**: la que descarga los datos de bolsa directamente de Yahoo Finance
- **networkx**: la que dibuja los grafos con flechas
- **statsmodels**: la que hace los tests de Granger
- **lingam**: la que implementa el algoritmo LiNGAM
- **plotly**: la que genera los grafos interactivos directamente en el notebook

---

## Sección 1 — Datos y Correlación de Pearson

Descargamos los precios diarios de los 5 activos para 24 años (2000–2024) y los convertimos en **retornos diarios** (cuánto subió o bajó cada día en porcentaje). Trabajamos con retornos y no con precios porque los precios casi siempre suben con el tiempo, lo que haría que todo pareciese relacionado aunque no lo estuviese.

Al final tenemos una tabla con **5.049 días** y **5 columnas** (una por activo).

En la misma celda calculamos la **correlación de Pearson** y la pintamos en un mapa de calor. La correlación nos dice si dos cosas se mueven juntas (rojo = positiva, azul = negativa), pero tiene un gran problema: es **simétrica**. La correlación de SP500 con VIX es exactamente igual que la de VIX con SP500. No tiene flechas. No hay dirección. No dice quién provoca a quién.

Por eso necesitamos algo más potente.

---

## Sección 2 — DAG con el Algoritmo PC

Aquí viene la parte más importante. Usamos el **algoritmo PC** (sus iniciales vienen de sus creadores: Peter Spirtes y Clark Glymour).

**¿Qué hace?**
1. Empieza conectando todo con todo
2. Va borrando conexiones donde detecta que dos variables son independientes
3. Orienta las flechas usando reglas lógicas

**El resultado que obtuvimos:**

```
SP500 → DXY     SP500 → TLT     SP500 ↔ VIX
DXY → TLT       TLT → DXY       VIX → GLD
```

**¿Qué significa en lenguaje normal?**
- SP500 y VIX se influyen mutuamente. Tiene sentido: cuando el mercado cae, el miedo sube; y cuando hay mucho miedo, el mercado puede caer.
- VIX también influye en el GLD. Cuando hay miedo, los inversores compran oro como refugio.
- El dólar (DXY) y los bonos (TLT) tienen una relación bidireccional.

En la misma celda también dibujamos la **matriz de adyacencia**: una tabla donde el 1 en la fila "VIX" y columna "GLD" significa VIX → GLD. Esta matriz es asimétrica, lo que confirma que tenemos dirección causal, no simple correlación.

---

## Sección 3 — Estabilidad temporal: 4 periodos en una figura

Aquí comprobamos si la estructura causal cambia según el momento del mercado. Dividimos los datos en **4 ventanas** y aplicamos PC en cada una, mostrando los 4 grafos juntos en una figura 2x2:

| Panel | Periodo | ¿Qué representa? |
|-------|---------|-----------------|
| Arriba-izquierda | 2000–2007 | Pre-crisis, mercados tranquilos |
| Arriba-derecha | 2010–2023 | Post-crisis, tipos a cero |
| Abajo-izquierda | 2013–2017 | Bull market, subidas continuas |
| Abajo-derecha | 2007–2009 | Bear market, crisis subprime |

**¿Por qué es interesante?** Si los 4 grafos son distintos, la estructura causal **no es estable en el tiempo**. En crisis (bear), los activos tienden a moverse todos juntos, lo que suele producir grafos más densos. En mercados alcistas (bull), las relaciones son más selectivas.

---

## Sección 4 — Comparación de los 3 algoritmos causales

Construimos el mismo DAG con tres métodos distintos, mostrados en una figura de 3 paneles:

### PC (panel izquierdo) — basado en restricciones
Borra conexiones donde detecta independencia estadística. Ya lo conocemos.

### HillClimbSearch (panel central) — basado en puntuación
Empieza con un grafo vacío y va añadiendo, quitando o cambiando flechas de una en una, eligiendo siempre el cambio que más mejore una puntuación llamada **BIC**. Es como escalar una montaña a ciegas: das el paso que más te sube, y cuando ya no puedes subir, te quedas ahí.

### LiNGAM (panel derecho) — basado en ICA
Aprovecha que los retornos financieros tienen "colas gordas" (los días extremos son más frecuentes de lo normal). Usa una técnica llamada **ICA** que detecta esas no-gaussianidades para orientar completamente todas las flechas, sin dejar ninguna sin dirección.

Si los tres coinciden en las aristas principales, tenemos mucha confianza. Si difieren mucho, los datos son ambiguos.

---

## Sección 5 — Causalidad temporal: Granger y PC lag-1

Esta sección combina dos análisis que se preguntan lo mismo de formas distintas: **¿el pasado de X ayuda a predecir el futuro de Y?**

### Granger (panel izquierdo)
El **test de Granger** (Premio Nobel de Economía) pregunta: ¿si añado los valores pasados del VIX, predigo mejor el SP500 que usando solo el pasado del SP500?

Si la respuesta es sí, decimos que el VIX "Granger-causa" al SP500. Mostramos los p-values de todos los pares en un mapa de calor. Las celdas rojas (p < 0.05) indican precedencia temporal significativa.

### PC lag-1 (panel derecho)
Construimos un dataframe con 10 columnas: las 5 originales más las mismas 5 retardadas un día (el valor de ayer). Aplicamos PC sobre esas 10 columnas. Cuando aparece una flecha de `VIX_lag1` hacia `SP500`, significa: "el VIX de ayer causalmente precede al SP500 de hoy", respetando la dirección del tiempo.

---

## Sección 6 — Robustez del PC: alpha y bootstrap

Esta sección agrupa dos análisis que validan si podemos confiar en el grafo que obtuvimos.

### Sensibilidad al alpha (panel izquierdo)
El algoritmo PC tiene un parámetro clave: `alpha`. Controla qué tan exigente es para eliminar conexiones. Alpha bajo (0.01) = pocos aristas; alpha alto (0.20) = muchas aristas. Probamos 4 valores y dibujamos cuántas aristas tiene el grafo en cada caso. Si el número cambia drásticamente, las conclusiones son frágiles.

### Bootstrap (panel derecho)
Cogemos nuestros 5.000 días y sacamos 20 muestras aleatorias con reemplazo. Aplicamos PC a cada muestra y anotamos qué flechas aparecen. Al final, cada celda del mapa de calor muestra qué fracción de las 20 muestras incluyó esa flecha. **1.0 = aparece siempre = muy robusta. 0.0 = nunca aparece = probablemente espuria.**

Usamos `np.random.seed(42)` para que los resultados sean reproducibles.

---

## Sección 7 — Visualización interactiva (plotly)

Todos los grafos anteriores son imágenes estáticas. Aquí generamos un **grafo interactivo directamente en el notebook** (sin generar ningún archivo externo) que permite:

- **Flechas dirigidas** que muestran claramente la dirección causal
- **Zoom y pan** con el ratón
- **Hover sobre cada nodo**: al pasar el cursor aparece el nombre del activo, quién le causa (in) y a quién causa (out)

---

## Sección 8 — Extra: El "Factor Mirage" (el peligro oculto del colisionador)

Esta es la parte más conceptualmente interesante.

### ¿Qué es un colisionador?

Imagina esta estructura:

```
SP500 → DXY ← TLT
```

Aquí DXY es un **colisionador**: tanto SP500 como TLT apuntan hacia él, pero SP500 y TLT no están directamente conectados. Son independientes si no miramos a DXY.

Pero en cuanto incluimos DXY en una regresión, **abrimos artificialmente un camino** entre SP500 y TLT. De repente parecen correlacionados, ¡aunque causalmente no lo estén directamente!

### El experimento

Predecimos el SP500 usando el GLD de dos formas:

- **Regresión sin DXY:** SP500 ~ GLD → coeficiente A
- **Regresión con DXY:** SP500 ~ GLD + DXY → coeficiente B (distinto)

Si DXY es un colisionador, incluirlo **distorsiona artificialmente** el coeficiente del GLD. Lo que parece "controlar por más variables = mejor análisis" es en realidad un error estadístico. El grafo causal nos dice exactamente qué variables incluir y cuáles no tocar.

---

## Resumen final: ¿Qué hemos aprendido?

| Lo que empezamos haciendo | Lo que conseguimos al final |
|---|---|
| Correlaciones simétricas (sin dirección) | Grafos con flechas (sabemos quién causa qué) |
| Un único análisis estático | 4 periodos comparados en una sola figura |
| Un único método (PC) | Tres algoritmos: PC, HillClimbSearch y LiNGAM |
| Granger y lag-1 por separado | Ambos en una figura conjunta de causalidad temporal |
| Sin verificar robustez | Alpha sweep + bootstrap en una figura conjunta |
| Gráfico estático del DAG | Grafo interactivo con flechas, zoom y hover en el notebook |
| Regresiones sin estructura causal | Identificación del Factor Mirage por colisionadores |

**La conclusión principal:** la correlación nos dice que las cosas se mueven juntas, pero no es suficiente para tomar decisiones causales. Los grafos causales (DAGs) nos dan la dirección de las influencias, nos permiten ver si esas influencias cambian con el tiempo y el régimen de mercado, y nos protegen de errores estadísticos. Comparar varios algoritmos, variar los parámetros y hacer bootstrap nos da confianza en qué partes del grafo son robustas y cuáles son frágiles.

---

*Este documento explica el notebook `notebook_causalidad.ipynb` del Taller B3-T3.*
