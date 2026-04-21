# Explicación del Trabajo: Causalidad entre Variables Financieras con Grafos

---

## ¿De qué trata esto en palabras normales?

Este trabajo intenta responder una pregunta muy concreta: **¿cuando sube el sector Tecnológico, eso hace que suba también el sector Financiero, o simplemente suelen moverse juntos por coincidencia?**

Es decir, no nos conformamos con saber que dos variables "van juntas". Queremos saber si una **provoca** a la otra. La herramienta que usamos para esto se llama **grafo causal** (o DAG), y es básicamente un diagrama con flechas que dice "esto influye en aquello".

---

## Los 6 sectores que estudiamos

Descargamos datos de 6 ETFs sectoriales de la bolsa americana, desde el año 2000 hasta 2024:

| Nombre | Ticker | ¿Qué es? |
|--------|--------|----------|
| **Financiero** | XLF | Bancos, aseguradoras y empresas de servicios financieros |
| **Energia** | XLE | Petroleras, gasistas y empresas de energía |
| **Tecnologia** | XLK | Empresas tecnológicas (software, hardware, semiconductores) |
| **Salud** | XLV | Farmacéuticas, hospitales y dispositivos médicos |
| **Industrial** | XLI | Manufactura, transporte y defensa |
| **Consumo** | XLY | Empresas de consumo discrecional (retail, ocio, autos) |

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

Descargamos los precios diarios de los 6 sectores para 24 años (2000–2024) y los convertimos en **retornos diarios** (cuánto subió o bajó cada día en porcentaje). Trabajamos con retornos y no con precios porque los precios casi siempre suben con el tiempo, lo que haría que todo pareciese relacionado aunque no lo estuviese.

Al final tenemos una tabla con **6.287 días** y **6 columnas** (una por sector).

En la misma celda calculamos la **correlación de Pearson** y la pintamos en un mapa de calor. La correlación nos dice si dos cosas se mueven juntas (rojo = positiva, azul = negativa), pero tiene un gran problema: es **simétrica**. La correlación de Tecnologia con Financiero es exactamente igual que la de Financiero con Tecnologia. No tiene flechas. No hay dirección. No dice quién provoca a quién.

Por eso necesitamos algo más potente.

> **¿Qué vemos en los datos reales?**
> Todos los sectores están positivamente correlacionados entre sí, lo cual tiene sentido: cuando la bolsa sube en general, suben todos. Las correlaciones más altas son Tecnologia–Consumo (0.81) y Tecnologia–Energia (0.79), y la más baja es Financiero–Salud (0.46). Esto nos dice que los sectores se mueven juntos, pero no quién arrastra a quién.

---

## Sección 2 — DAG con el Algoritmo PC

Aquí viene la parte más importante. Usamos el **algoritmo PC** (sus iniciales vienen de sus creadores: Peter Spirtes y Clark Glymour).

**¿Qué hace?**
1. Empieza conectando todo con todo
2. Va borrando conexiones donde detecta que dos variables son independientes
3. Orienta las flechas usando reglas lógicas

En la misma celda también dibujamos la **matriz de adyacencia**: una tabla donde el 1 en la fila "Tecnologia" y columna "Financiero" significa Tecnologia → Financiero. Esta matriz es asimétrica, lo que confirma que tenemos dirección causal, no simple correlación.

> **¿Qué vemos en los datos reales?**
> El DAG resultante tiene 13 aristas. El sector **Tecnologia** es el nodo más central: recibe influencia de Energia, Financiero, Consumo y Salud, y a la vez influye sobre todos ellos. El **Consumo** también actúa como motor: causa a Salud, Energia, Industrial y Tecnologia. En cambio, el sector **Industrial** no aparece como causante de ningún otro sector en este grafo global. La estructura es densa, reflejando la alta interconexión de los sectores americanos durante 24 años de datos.

---

## Sección 3 — Estabilidad temporal: 4 periodos en una figura

Aquí comprobamos si la estructura causal cambia según el momento del mercado. Dividimos los datos en **4 ventanas** y aplicamos PC en cada una, mostrando los 4 grafos juntos en una figura 2x2:

| Panel | Periodo | ¿Qué representa? |
|-------|---------|-----------------|
| Arriba-izquierda | 2000–2007 | Pre-crisis, mercados tranquilos |
| Arriba-derecha | 2010–2023 | Post-crisis, tipos a cero |
| Abajo-izquierda | 2013–2017 | Bull market, subidas continuas |
| Abajo-derecha | 2007–2009 | Bear market, crisis subprime |

> **¿Qué vemos en los datos reales?**
> La estructura causal **sí cambia** significativamente según el régimen de mercado:
> - **Pre-crisis (2000–2007):** Consumo y Energia son los principales motores, con Financiero también activo como causa. Salud tiene un papel causal relevante sobre Industrial.
> - **Post-crisis (2010–2023):** La estructura se simplifica. Financiero y Consumo siguen causando a otros sectores. Salud gana protagonismo causal sobre Industrial y Tecnologia.
> - **Bull market (2013–2017):** Tecnologia explota como hub central: causa a Financiero, Consumo, Salud, Energia e Industrial simultáneamente — reflejo del boom tecnológico.
> - **Bear/Crisis (2007–2009):** El grafo se vuelve el más denso de todos (16 aristas). Financiero, que en otros periodos recibe influencias, aquí pasa a **causar** a Salud, Industrial y Tecnologia — reflejo del papel del sistema financiero como detonante de la crisis subprime.

---

## Sección 4 — Comparación de los 3 algoritmos causales

Construimos el mismo DAG con tres métodos distintos, mostrados en una figura de 3 paneles:

### PC (panel izquierdo) — basado en restricciones
Borra conexiones donde detecta independencia estadística. Ya lo conocemos.

### HillClimbSearch (panel central) — basado en puntuación
Empieza con un grafo vacío y va añadiendo, quitando o cambiando flechas de una en una, eligiendo siempre el cambio que más mejore una puntuación llamada **BIC**. Es como escalar una montaña a ciegas: das el paso que más te sube, y cuando ya no puedes subir, te quedas ahí.

### LiNGAM (panel derecho) — basado en ICA
Aprovecha que los retornos financieros tienen "colas gordas" (los días extremos son más frecuentes de lo normal). Usa una técnica llamada **ICA** que detecta esas no-gaussianidades para orientar completamente todas las flechas, sin dejar ninguna sin dirección.

> **¿Qué vemos en los datos reales?**
> Los tres algoritmos coinciden en un punto clave: **Tecnologia es un hub causal central** que influye sobre el resto de sectores. Pero difieren en los detalles:
> - **HillClimbSearch** otorga un papel protagonista a **Industrial** como causa de Tecnologia, Consumo, Salud y Energia — algo que PC no detecta.
> - **LiNGAM** orienta completamente todas las flechas y sitúa a **Industrial** como el gran motor del sistema, causando a Financiero, Energia, Salud y Consumo. También detecta que Tecnologia causa a todos los demás. El acuerdo entre PC y LiNGAM en Tecnologia → {Financiero, Energia, Salud, Industrial, Consumo} da mucha confianza a esas relaciones.
> - La mayor discrepancia está en el papel de **Financiero**: para PC recibe y emite influencia sobre Tecnologia, para HillClimb es más causante que receptor, y para LiNGAM es principalmente receptor.

---

## Sección 5 — Causalidad temporal: Granger y PC lag-1

Esta sección combina dos análisis que se preguntan lo mismo de formas distintas: **¿el pasado de X ayuda a predecir el futuro de Y?**

### Granger (panel izquierdo)
El **test de Granger** (Premio Nobel de Economía) pregunta: ¿si añado los valores pasados de Tecnologia, predigo mejor el Financiero que usando solo el pasado del Financiero?

Si la respuesta es sí, decimos que Tecnologia "Granger-causa" a Financiero. Mostramos los p-values de todos los pares en un mapa de calor. Las celdas rojas (p < 0.05) indican precedencia temporal significativa.

### PC lag-1 (panel derecho)
Construimos un dataframe con 12 columnas: los 6 sectores originales más los mismos 6 retardados un día (el valor de ayer). Aplicamos PC sobre esas 12 columnas. Cuando aparece una flecha de `Tecnologia_lag1` hacia `Financiero`, significa: "el sector tecnológico de ayer causalmente precede al sector financiero de hoy", respetando la dirección del tiempo.

> **¿Qué vemos en los datos reales?**
> El test de Granger revela una red de precedencias temporales muy rica. Los resultados más llamativos:
> - **Salud es el sector con más poder predictivo**: su pasado predice significativamente a Financiero, Energia, Tecnologia, Industrial y Consumo (todos con p < 0.05). Esto es sorprendente — el sector salud actúa como adelantado del resto.
> - **Consumo también predice a todos** los demás sectores (p < 0.05 en los 5 pares).
> - **Financiero predice a todos** los sectores, confirmando que los mercados financieros reaccionan primero y el resto le sigue.
> - La única relación débil es Energia → Salud (p = 0.058) y Tecnologia → Energia (p = 0.118), que no son significativas.

---

## Sección 6 — Robustez del PC: alpha y bootstrap

Esta sección agrupa dos análisis que validan si podemos confiar en el grafo que obtuvimos.

### Sensibilidad al alpha (panel izquierdo)
El algoritmo PC tiene un parámetro clave: `alpha`. Controla qué tan exigente es para eliminar conexiones. Alpha bajo (0.01) = pocos aristas; alpha alto (0.20) = muchas aristas. Probamos 4 valores y dibujamos cuántas aristas tiene el grafo en cada caso. Si el número cambia drásticamente, las conclusiones son frágiles.

### Bootstrap (panel derecho)
Cogemos nuestros 6.287 días y sacamos 20 muestras aleatorias con reemplazo. Aplicamos PC a cada muestra y anotamos qué flechas aparecen. Al final, cada celda del mapa de calor muestra qué fracción de las 20 muestras incluyó esa flecha. **1.0 = aparece siempre = muy robusta. 0.0 = nunca aparece = probablemente espuria.**

Usamos `np.random.seed(42)` para que los resultados sean reproducibles.

> **¿Qué vemos en los datos reales?**
> - **Alpha sweep:** El grafo tiene 14 aristas con alpha=0.01 y baja a 13 aristas con alpha=0.05 o superior. La estructura es **muy estable**: cambiar el umbral de exigencia no hace colapsar ni explotar el grafo, lo que da confianza en los resultados.
> - **Bootstrap:** Las aristas más robustas (frecuencia 1.0 en 20 muestras) son **Financiero → Tecnologia** y **Consumo → Energia** — aparecen en el 100% de las muestras. Con frecuencia ≥ 0.90 también están Tecnologia → {Consumo, Energia, Industrial} (0.95 cada una). En cambio, aristas como Energia → Financiero o Salud → Energia aparecen solo en 1 de cada 20 muestras (0.05), y deben considerarse frágiles o espurias.

---

## Sección 7 — Visualización interactiva (plotly)

Todos los grafos anteriores son imágenes estáticas. Aquí generamos un **grafo interactivo directamente en el notebook** (sin generar ningún archivo externo) que permite:

- **Flechas dirigidas** que muestran claramente la dirección causal
- **Zoom y pan** con el ratón
- **Hover sobre cada nodo**: al pasar el cursor aparece el nombre del sector, quién le causa (in) y a quién causa (out)

> **¿Qué vemos en los datos reales?**
> Al hacer hover sobre el nodo **Tecnologia** se ve que tiene 4 causas entrantes (Energia, Financiero, Consumo, Salud) y 5 efectos salientes, confirmando visualmente su posición como el nodo más conectado del DAG. El nodo **Industrial** solo aparece como destino de flechas — ningún otro nodo recibe influencia de él en el grafo global.

---

## Sección 8 — Extra: El "Factor Mirage" (el peligro oculto del colisionador)

Esta es la parte más conceptualmente interesante.

### ¿Qué es un colisionador?

Imagina esta estructura:

```
Consumo → Tecnologia ← Salud
```

Aquí Tecnologia es un **colisionador**: tanto Consumo como Salud apuntan hacia él, pero Consumo y Salud no están directamente conectados. Son independientes si no miramos a Tecnologia.

Pero en cuanto incluimos Tecnologia en una regresión, **abrimos artificialmente un camino** entre Consumo y Salud. De repente parecen correlacionados, ¡aunque causalmente no lo estén directamente!

### El experimento

Predecimos Consumo usando Salud de dos formas:

- **Regresión sin Tecnologia:** Consumo ~ Salud → coeficiente = **0.6527**
- **Regresión con Tecnologia:** Consumo ~ Salud + Tecnologia → coeficiente = **0.2781**

> **¿Qué vemos en los datos reales?**
> Al incluir Tecnologia como variable de control, el coeficiente de Salud sobre Consumo **cae de 0.65 a 0.28** — una diferencia de -0.37. Es decir, lo que parecía una influencia fuerte de Salud sobre Consumo se reduce casi a la mitad cuando "controlamos" por Tecnologia.
>
> Esto no es porque Tecnologia mejore el modelo. Es porque Tecnologia **es un colisionador**: al incluirlo, abrimos artificialmente el camino Consumo ↔ Salud a través de Tecnologia, distorsionando el coeficiente. Un analista sin el grafo causal podría pensar que "controlar por más variables es mejor" y caer en este error. **El grafo causal nos dice exactamente qué variables incluir y cuáles no tocar.**

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

**La conclusión principal:** la correlación nos dice que los sectores se mueven juntos, pero no es suficiente para tomar decisiones causales. Los grafos causales (DAGs) nos dan la dirección de las influencias entre sectores, nos permiten ver si esas influencias cambian con el tiempo y el régimen de mercado, y nos protegen de errores estadísticos. El sector **Tecnologia** emerge como el hub causal más robusto del sistema, con **Financiero → Tecnologia** y **Consumo → Energia** como las relaciones más estables a lo largo del tiempo.

---

*Este documento explica el notebook `notebook_causalidad.ipynb` del Taller B3-T3.*
