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

## Paso 0 y 1 — Preparar las herramientas

Antes de hacer nada, instalamos las librerías de Python que vamos a necesitar. Es como abrir la caja de herramientas antes de ponerse a trabajar.

Las más importantes son:
- **pgmpy**: la que sabe construir grafos causales
- **yfinance**: la que descarga los datos de bolsa directamente de Yahoo Finance
- **networkx**: la que dibuja los grafos con flechas

---

## Paso 2 — Obtener los datos

Descargamos los precios diarios de los 5 activos para 24 años (2000–2024).

**Importante:** no trabajamos con los precios directamente. Los convertimos en **retornos diarios** (cuánto subió o bajó cada día en porcentaje). Por ejemplo, si el SP500 valía 1000 ayer y hoy vale 1010, el retorno es +1%.

¿Por qué hacemos esto? Porque los precios suben casi siempre con el tiempo (por inflación, crecimiento...), lo que hace que todo parezca relacionado aunque no lo esté. Los retornos eliminan ese efecto y nos dan una señal más limpia.

Al final tenemos una tabla con **5.049 días** y **5 columnas** (una por activo).

---

## Paso 3 — La correlación de Pearson (el punto de partida, y su problema)

Lo primero que hacemos es calcular la **correlación de Pearson** entre todas las variables.

**¿Qué es la correlación?** Es un número entre -1 y +1 que dice si dos cosas se mueven juntas:
- **+1**: cuando una sube, la otra también sube siempre
- **-1**: cuando una sube, la otra siempre baja
- **0**: no se mueven juntas para nada

Pintamos esto en un mapa de calor (heatmap) donde los colores rojos indican correlación positiva y los azules negativa.

**El gran problema de la correlación:** es **simétrica**. Eso significa que la correlación de SP500 con VIX es exactamente la misma que la de VIX con SP500. No nos dice quién va primero. No tiene flechas. No hay dirección. Es como decir "los paraguas y la lluvia aparecen juntos" sin saber si la lluvia causa que saques el paraguas o si el paraguas causa la lluvia.

Por eso necesitamos algo más potente.

---

## Paso 4 — El algoritmo PC: encontrar las flechas causales

Aquí viene la parte más importante del trabajo. Usamos el **algoritmo PC** (sus iniciales vienen de sus creadores: Peter Spirtes y Clark Glymour).

**¿Qué hace el algoritmo PC?**

1. **Empieza conectando todo con todo** (como si todas las variables influyesen en todas)
2. Luego va **borrando conexiones** donde detecta que dos variables son independientes, incluso condicionando por las demás
3. Finalmente **orienta las flechas** usando unas reglas lógicas

Para decidir si dos variables son independientes, hace un test estadístico llamado **test de independencia condicional** (aquí usamos `pearsonr`). Básicamente pregunta: "¿si ya sé lo que hace el VIX, saber también lo que hace el SP500 me da información nueva sobre el GLD?" Si la respuesta es no, elimina esa conexión.

**El resultado que obtuvimos:**

```
SP500 → DXY
SP500 → TLT
SP500 → VIX
DXY → SP500
DXY → TLT
TLT → DXY
VIX → SP500
VIX → GLD
```

**¿Qué significa esto en lenguaje normal?**

- El SP500 y el VIX se **influyen mutuamente** (flechas en los dos sentidos). Tiene sentido: cuando el mercado cae, el miedo sube; y cuando hay mucho miedo, el mercado puede caer.
- El VIX también influye en el GLD. Cuando hay miedo, los inversores compran oro como refugio.
- El dólar (DXY) y los bonos (TLT) tienen una relación bidireccional: se afectan el uno al otro.
- El SP500 influye en el dólar y los bonos.

Este grafo ya tiene **flechas**, no es simétrico. Eso es lo que lo hace más valioso que la simple correlación.

---

## Paso 5 — Visualizar el grafo y la matriz de adyacencia

Dibujamos el DAG (grafo acíclico dirigido) de dos formas:

**1. Como un diagrama con flechas:** los nodos son los 5 activos y las flechas indican quién influye en quién.

**2. Como una matriz numérica (matriz de adyacencia):** una tabla donde si hay un 1 en la fila "VIX" y columna "GLD", significa que VIX → GLD. Esta matriz es **asimétrica** (no es igual arriba que abajo de la diagonal), lo que confirma que tenemos dirección causal, no simple correlación.

---

## Paso 6 — Variación 1: ¿Cambia la causalidad antes y después de la crisis de 2008?

Aquí dividimos los datos en dos épocas:

- **Pre-crisis:** 2000 a 2007
- **Post-crisis:** 2010 a 2023 (excluimos 2008–2009 para no "contaminar" ninguna de las dos épocas)

Aplicamos el algoritmo PC por separado en cada período y comparamos los grafos.

**¿Por qué es interesante esto?** La crisis financiera de 2008 fue un evento brutal que cambió cómo funcionan los mercados. Es razonable pensar que las relaciones causales entre activos no son las mismas en tiempos tranquilos que en tiempos de pánico o en la era posterior al rescate bancario con tipos de interés casi a cero.

Si los grafos pre y post-crisis son distintos, eso nos dice que la **estructura causal no es estable en el tiempo**, lo cual es una información muy valiosa para cualquier inversor o analista.

---

## Paso 7 — Variación 2: Dos métodos para encontrar el grafo causal

Aquí comparamos dos formas distintas de construir el DAG:

### Método 1: Algoritmo PC (basado en restricciones)
Como ya explicamos: borra conexiones donde detecta independencia estadística y luego orienta las flechas.

### Método 2: HillClimbSearch (basado en puntuación)
Este método funciona de forma completamente diferente:
- Empieza con un grafo vacío (sin ninguna conexión)
- Va añadiendo, quitando o cambiando flechas **de una en una**
- En cada paso, elige el cambio que más mejore una puntuación llamada **BIC** (Bayesian Information Criterion), que premia los grafos que explican bien los datos pero sin ser demasiado complicados
- Para cuando no puede mejorar más

Es como escalar una montaña a ciegas: das el paso que más te sube, y cuando ya no puedes subir más, te quedas ahí.

**¿Por qué comparar ambos?** Porque dos métodos válidos pueden dar grafos distintos. Si coinciden en las aristas más importantes, tenemos más confianza en que esas relaciones son reales. Si difieren mucho, nos dice que los datos no son suficientemente claros para decidir la estructura causal con certeza.

---

## Paso 8 — Extra: El "Factor Mirage" (el peligro oculto del colisionador)

Esta es la parte más conceptualmente interesante del trabajo.

### ¿Qué es un colisionador?

Imagina esta estructura causal:

```
SP500 → DXY ← TLT
```

Aquí DXY es un **colisionador**: tanto el SP500 como el TLT apuntan hacia él, pero SP500 y TLT no están directamente conectados entre sí.

En este caso, **SP500 y TLT son independientes** si no miramos a DXY. Pero en cuanto incluimos DXY en nuestra análisis (por ejemplo, en una regresión), abrimos un camino entre SP500 y TLT que antes estaba bloqueado. De repente parecen correlacionados, ¡aunque causalmente no lo están directamente!

### El experimento que hacemos

Probamos a predecir el SP500 usando el GLD (oro):

**Regresión 1:** SP500 ~ GLD *(sin controlar por DXY)*
- Coeficiente del GLD: un valor concreto

**Regresión 2:** SP500 ~ GLD + DXY *(controlando por DXY)*
- Coeficiente del GLD: un valor diferente

Si el DXY es un colisionador en el grafo causal, incluirlo en la regresión **distorsiona artificialmente** el coeficiente del GLD. Lo que parecería ser "controlar por más variables = mejor análisis" es en realidad un **error estadístico**.

### ¿Por qué esto importa en la práctica?

En finanzas, los analistas suelen añadir muchas variables de control "por si acaso". Pero si alguna de esas variables es un colisionador, están introduciendo sesgos espurios en sus modelos. El grafo causal nos dice exactamente **qué variables debemos incluir** y cuáles no debemos tocar.

---

## Resumen final: ¿Qué hemos aprendido?

| Lo que empezamos haciendo | Lo que conseguimos al final |
|---|---|
| Correlaciones simétricas (no sabemos quién causa qué) | Grafos con flechas (sabemos las direcciones) |
| Un único análisis con todos los datos | Comparación pre-crisis vs post-crisis |
| Un único método | Dos métodos comparados (PC y HillClimbSearch) |
| Regresiones sin pensar en la estructura causal | Identificación del "Factor Mirage" por colisionadores |

**La conclusión principal:** la correlación nos dice que las cosas se mueven juntas, pero no es suficiente para tomar decisiones causales. Los grafos causales (DAGs) nos dan la dirección de las influencias, nos permiten ver si esas influencias cambian con el tiempo, y nos protegen de errores estadísticos como el sesgo de colisionador.

---

*Este documento explica el notebook `notebook_causalidad.ipynb` del Taller B3-T3.*
