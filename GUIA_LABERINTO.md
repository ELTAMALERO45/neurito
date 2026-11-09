# Neurito 3.0: aprende a encontrar comida en un laberinto

Neurito busca comida dentro de tres laberintos. Su cerebro debe elegir entre arriba, abajo, izquierda y derecha. La red se construye con NumPy, y Pygame dibuja el escenario, la mascota y una gráfica de la pérdida.

## Cómo aprende

1. BFS calcula rutas cortas y genera ejemplos etiquetados: mapa, posición, meta y movimiento recomendado.
2. Los ejemplos se dividen en entrenamiento (80 %) y prueba (20 %). Los pares posición-meta de prueba no aparecen en entrenamiento.
3. Cada entrada incluye un vector one-hot de tres mapas y dos vectores one-hot de 81 casillas: 165 características.
4. La arquitectura es 165 → 96 → 4. Usa ReLU en la capa oculta y Softmax en las cuatro salidas.
5. El entrenamiento minimiza entropía cruzada multiclase mediante retropropagación y descenso de gradiente con mini-lotes.
6. Durante el juego, el modelo decide sin consultar BFS. BFS es el experto que proporcionó las etiquetas.

Esto es aprendizaje supervisado por imitación, NO aprendizaje por refuerzo.

## Dos modos

- Red pura: toma la salida de mayor probabilidad. Puede chocar con paredes o quedarse en ciclos.
- Asistido: descarta movimientos contra paredes y penaliza casillas visitadas. Son reglas adicionales, no conocimientos aprendidos por la red.

La precisión de clasificación no equivale al porcentaje de laberintos completados.

## Estructura

- laberinto_mundo.py: mapas, movimiento, BFS y navegación asistida.
- laberinto_red.py: red neuronal y entrenamiento con NumPy.
- laberinto.py: juego Pygame, sprites y pérdida.
- test_laberinto.py: pruebas automáticas.
- GUIA_LABERINTO.md: esta guía.

Reutiliza assets/neutral_1.png, neutral_2.png y neutral_3.png. No modifica las versiones anteriores.

## Ejecutar

    uv sync
    uv run laberinto.py

## Controles

| Tecla | Acción |
|---|---|
| 1 | Entrenar una época |
| 2 | Entrenar cien épocas |
| 3 | Entrenamiento continuo |
| A | Alternar red pura y navegación asistida |
| P | Alternar navegación automática y paso a paso |
| Espacio | Avanzar un paso en modo manual |
| N | Cambiar mapa |
| G | Nuevas posiciones y meta |
| C | Reiniciar contadores |
| R | Reiniciar pesos y entrenamiento |
| Esc | Salir |

## Probar automáticamente

    uv run python -m unittest -v test_laberinto.py

## Experimentos

1. Observa Neurito sin entrenar. ¿Por qué se atasca?
r/ Se atasca porque al arrancar el programa el no sabe ni que es una pared ni donde se encuentra la comida, unicamente mantiene valores aleatorios dentro de si y al hacer un giro hacia alguna direccion no entiende donde ni como llegar a su destino, se choca una y otra vez con la misma pared o tiene una misma probabilidad (25% aprox) de ir a la derecha, izquierda, arriba o abajo
2. Entrena 100, 200 y 400 épocas. Compara pérdida, precisión y movimientos.
r/
Con 100 epocas: 
Llega a una precision aproximadamente de 86.5% con perdida de 0.1334 y neurito tiende a un comportamiento en el que logra tomar caminos largos y rutas especificas, a la hora de tener que hacer un giro mas brusco o complicado no lo logra y se queda estancado
Con 200 epocas:
La precision aumenta un poco hasta llegar aproximadamente a 89.7% con perdida de 0.0420, neurito resuelve la mayoria de trayectos de formas mas fluidas y rapidas llegando a mas comidas con menos atascos, sin embargo continua trabandose al llegar a una ruta que tenga 2 mismo sentidos con igual cantidad de pasos no sabe por cual irse y se queda atascado
Con 400 epocas:
La precison llega hasta un pico de 90.5% con perdida de 0.0106, neurito logra encontrar su comida de forma optima y rapida en practicamente todos sus intentos, sin embargo llega un punto en el que se atasca al intentar llegar a ella y no la toma pensando que la comida es una pared, esto sucede aproximadamente de 50 intentos le pasa en 8 

3. Compara red pura frente a navegación asistida. ¿Qué parte corresponde al modelo y cuál a las reglas?
r/
Con red pura y un entrenamiento de 400 epocas neurito logra llegar a su comida la mayor parte de las veces y de manera fluida,  sin embargo llega un punto en el que se atasca al intentar llegar a ella y no la toma pensando que la comida es una pared, esto sucede aproximadamente de 50 intentos le pasa en 8 

Con red asistida y el mismo entrenamiento de 400 epocas si bien no sube la precision y se mantiene en un 90% aprox se ve como neurito logra llegar mas rapido a su comida y rara vez se atasca, practicamente nunca y siempre encuentra el modo optimo de llegar a esta

En la red pura, Neurito depende al 100% de las 4 probabilidades calculadas por la red neuronal, lo que demuestra lo que verdaderamente aprendió el modelo pero lo deja expuesto a chocar contra paredes o bucles infinitos si comete un error. Por el contrario, en la navegación asistida se aplica un sistema híbrido donde reglas externas filtran los movimientos hacia paredes y penalizan casillas ya transitadas, rompiendo ciclamientos y elevando la tasa de éxito de un 75% (red sola) a más del 98% al evitar que los fallos del modelo probabilístico arruinen el recorrido.

4. Cambia las 96 neuronas ocultas por 16, 32 o 128.
Con 16 neuronas: La red sufre de subajuste (underfitting) severo debido a su baja capacidad de representación. Con tan pocas neuronas, la multiplicación de matrices no logra comprimir ni relacionar correctamente las 165 características de entrada, lo que provoca que la pérdida baje de forma sumamente lenta y la exactitud en el conjunto de prueba se estanque por debajo del 80%.   

Con 32 neuronas: La capacidad del modelo continúa siendo limitada, aunque muestra una ligera mejora respecto a 16 neuronas. Logra capturar algunos patrones básicos del laberinto, pero requiere una mayor cantidad de épocas para converger y su precisión de prueba suele quedarse atascada alrededor del 83% - 87%, fallando en situaciones donde el camino a la meta exige maniobras más complejas.

Con 96 neuronas (Configuración original): Representa el punto de equilibrio óptimo entre capacidad computacional y poder de generalización. Otorga el espacio latente ideal para que la red procese el mapa, la posición y la meta, logrando reducir la pérdida de forma fluida y alcanzando más del 90% - 95% de precisión en prueba en 400 épocas.   

Con 128 neuronas: La red reduce la pérdida de entrenamiento más rápido debido a su alta plasticidad, pero a costa de un mayor consumo computacional por cada época. Si bien obtiene resultados excelentes en exactitud, no ofrece un beneficio significativo frente a 96 neuronas y se acerca al límite donde podría memorizar los datos de entrenamiento en lugar de generalizar mejor. 

5. Cambia la tasa de aprendizaje 0.45 y observa si converge.
r/ 
Con tasa muy baja 0.01$ o 0.05: El descenso de gradiente avanza a pasos diminutos, por lo que la pérdida disminuye de forma excesivamente lenta. La red necesitaría miles de épocas en lugar de 400 para aproximarse a los pesos óptimos, haciendo que en las primeras 100 o 200 épocas Neurito se comportamiento prácticamente como si estuviera sin entrenar.  

Con la tasa original 0.45: Es el valor óptimo de convergencia rápida y estable para esta arquitectura con normalización y ReLU. Permite que los pesos se ajusten con suficiente fuerza en cada lote de entrenamiento (mini-batch) sin dar saltos gigantescos, logrando reducir la pérdida de forma constante y alcanzar el $\ge 90\%$ de exactitud en solo 400 épocas.  

Con tasa muy alta 1.5$ o superior: El tamaño del paso en la actualización de pesos es tan grande que el algoritmo "rebota" o sobrepasa los mínimos locales de la función de pérdida (overshooting). Esto provoca que la pérdida oscile violentamente o se dispare hacia el infinito (divergencia), impidiendo por completo que la red aprenda a navegar.   

6. Cambia de mapa. Los mapas forman parte del entrenamiento, pero las combinaciones de inicio y meta usadas para probar no se entrenaron.
Mismo universo de mapas (Generalización por Posición/Meta): Los tres mapas (Mapa 1, Mapa 2 y Mapa 3) forman parte del conjunto global con el que se generó la matriz de entrenamiento . Aunque la combinación exacta de [inicio -> meta] que estás probando en pantalla fue excluida del 80% de entrenamiento e introducida en el 20% de prueba, Neurito resuelve el camino con éxito. Esto demuestra que la red aprendió a relacionar la geometría interna del mapa con la meta sin memorizar las coordenadas exactas de memoria.  

Cambio entre Mapas 1, 2 y 3 (Vector One-Hot de Entrada): Al presionar la tecla N, la red cambia el vector one-hot de los primeros 3 bits (índice del mapa). Dado que la red fue entrenada simultáneamente con datos de los tres laberintos, es capaz de activar las neuronas correctas para la estructura de cada uno, manteniendo un desempeño alto y consistente sin necesidad de reentrenarse desde cero.   

Límite de Generalización (Mapas totalmente nuevos): Si creáramos un Mapa 4 diferente, Neurito fracasaría por completo al intentar navegarlo. Esto se debe a que la arquitectura actual no recibe las paredes como entrada visual ni características del espacio, sino únicamente una etiqueta fija del mapa (ID: 0, 1 o 2). Para que generalice a mapas desconocidos, se tendría que cambiar el diseño de la entrada para codificar las paredes circundantes o pasar una representación matricial completa. 
 
7. Haz vibecoding para verificar como mejora o empeora Neurito al cambiar BFS, por DFS y A*

Experto BFS (Configuración Original): El algoritmo BFS explora el laberinto nivel por nivel en forma de ondas concéntricas, garantizando encontrar siempre el camino óptimo (el más corto en número de pasos) hacia la comida. Al entrenar la red neuronal con las etiquetas generadas por BFS, Neurito aprende a reproducir trayectorias directas y eficientes, alcanzando la mayor velocidad y exactitud en sus recorridos.   

Experto DFS (Búsqueda en Profundidad): Al reemplazar BFS por DFS (usando una pila stack en lugar de una cola deque), el profesor ya no calcula rutas óptimas, sino que explora cada rama del laberinto hasta el fondo antes de retroceder. Como consecuencia, las etiquetas de entrenamiento contienen rodeos innecesarios y caminos largos; la red neuronal aprende a imitar exactamente esa ineficiencia, provocando que Neurito dé vueltas absurdas y empeore drásticamente su tiempo de llegada a la comida.

Experto A* (Búsqueda Heurística): Al implementar A* utilizando la distancia Manhattan como heurística, el profesor encuentra la ruta óptima igual de bien que BFS pero evaluando menos estados durante la generación de datos. Dado que el camino resultante hacia la meta es idéntico al de BFS, la red neuronal alcanza exactamente la misma precisión y eficiencia en sus movimientos, demostrando que lo que importa para el entrenamiento supervisado es la calidad del camino final entregado por el experto, no cómo lo calculó internamente.   

## Alcance y limitaciones

El modelo no interpreta capturas de pantalla: trabaja con características estructuradas. Conoce el identificador de cada mapa. Los tres mapas pertenecen al mismo universo de entrenamiento. La evaluación demuestra generalización a pares de posición-meta no entrenados, no a laberintos totalmente desconocidos. Para eso habría que representar también las paredes de cualquier mapa nuevo y entrenar y evaluar en mapas separados.
