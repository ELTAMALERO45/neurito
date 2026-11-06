# Neurito 2.0: aprende a saltar obstáculos

Neurito aprende **cuándo saltar** ante obstáculos que se aproximan con diferentes velocidades. El cerebro se implementa manualmente con NumPy y el aprendizaje puede observarse en Pygame.

## ¿Qué aprende?

La red recibe tres características normalizadas: distancia al obstáculo, velocidad y tiempo estimado hasta alcanzarlo (distancia / velocidad). La última es una característica derivada, no una observación independiente.

La arquitectura es **3 → 12 → 1**: doce neuronas ocultas con ReLU y una salida con sigmoide. La última devuelve una puntuación entre cero y uno que, con el umbral 0.5, determina si Neurito debe saltar cuando está en el suelo.

## ¿Cómo aprende?

Aplicamos **aprendizaje supervisado**, no aprendizaje por refuerzo. Creamos datos sintéticos que incluyen ejemplos donde es conveniente saltar y otros donde debe esperar. Para generar las etiquetas, definimos una ventana de referencia: saltar cuando el tiempo estimado está entre **7 y 13 fotogramas**. No es una regla aprendida por recompensa ni garantiza evitar choques en todas las situaciones.

La red ajusta sus pesos a partir de la **entropía cruzada binaria**, la **retropropagación** y el **descenso de gradiente**:

`W_nuevo = W_anterior - tasa_de_aprendizaje * gradiente`

El conjunto de entrenamiento equilibra aproximadamente las dos decisiones. De otra manera, una red podría aparentar buena precisión diciendo casi siempre «esperar».

La precisión se mide sobre ejemplos distintos de los utilizados en el entrenamiento. **Una precisión alta no garantiza un buen resultado en el juego**, que también depende de la física del salto.

## Archivos

```text
Neurito/
├── main.py                 # Mascota original: comer, dormir, jugar
├── red-sencilla.py         # Ejemplo de una neurona
├── salto_modelo.py         # Red neuronal manual y datos de entrenamiento
├── salto_juego.py          # Física y detección de colisiones
├── saltar.py               # Juego y visualización del entrenamiento
├── GUIA_SALTO.md
└── assets/
    └── jugar_1.png        # Reutilizado en el nuevo juego
```

## Ejecución

```bash
uv sync
uv run saltar.py
```

No se necesitan dependencias nuevas: se reutilizan NumPy y Pygame.

## Controles

| Tecla | Función |
|---|---|
| `1` | Entrenar una época |
| `2` | Entrenar cien épocas |
| `3` | Activar o pausar entrenamiento continuo |
| `A` | Alternar control automático y manual |
| `Espacio` | Saltar manualmente |
| `C` | Reiniciar pista y contadores |
| `R` | Reiniciar pesos y contadores |
| `Esc` | Salir |

## Experimentos

1. Prueba el juego sin entrenar. Anota colisiones y obstáculos esquivados.
-Neurito no reacciona o salta en momentos completamente arbitrarios. Los pesos de la red están inicializados aleatoriamente, por lo que la salida de la sigmoide oscila en valores que no tienen relación con la física del juego.

Pérdida (Loss): Alta (alrededor de 0.693 o superior).

Precisión (Accuracy): ~50% (equivalente a adivinar al azar).

Desempeño en juego: Choque casi inmediato en el primer o segundo obstáculo.

2. Entrena de cien en cien épocas; compara la pérdida, la precisión y el comportamiento.
De 0 a 100 épocas: La pérdida cae drásticamente. Neurito empieza a entender la relación entre distancia/velocidad y el salto. La precisión sube rápidamente al 80% - 85%.

De 100 a 500 épocas: La pérdida disminuye de forma más suave y suave. La precisión de prueba se estabiliza por encima del 92% - 96%.

Comportamiento en juego: Neurito esquiva con éxito la mayoría de los obstáculos a velocidades estándar. Si el entrenamiento es muy continuo, la red refina su umbral de decisión en el momento justo del despegue.

3. Cambia `ocultas=12` por `ocultas=3` o `ocultas=24`; compara generalización.

Con ocultas = 3 (Red pequeña): Presenta subajuste (underfitting). La red es demasiado simple para entender la combinación de distancia y velocidad al mismo tiempo. Su precisión de prueba no sube del 75% - 85% y choca seguido porque no logra calcular bien la frontera de decisión.

Con ocultas = 12 (Configuración base): Es el punto de equilibrio óptimo. Tiene las neuronas necesarias para aprender a combinar las 3 entradas sin memorizar ruido. Alcanza más del 95% de precisión y responde bien a distintas velocidades.

Con ocultas = 24 (Red grande): Corre riesgo de sobreajuste (overfitting). Aprende muy rápido el conjunto de datos de entrenamiento, pero se vuelve innecesariamente compleja. Aunque marca una precisión de prueba altísima en los datos sintéticos, puede fallar en el juego real ante situaciones límite que no vio exactamente igual en el entrenamiento.

4. Modifica la tasa de aprendizaje `0.8` a `0.01` o `1.2` y estudia la convergencia.
Tasa baja (0.01):

Convergencia: Extremadamente lenta. Se necesitan miles de épocas (presionar muchas veces 2) para ver reducir la pérdida significativamente.

Estabilidad: Muy alta; avanza a pasos pequeños sin oscilar.

Tasa base (0.8):

Convergencia: Rápida y fluida en pocas decenas de épocas. Adecuada para este problema sintético.

Tasa alta (1.2):

Convergencia: Inestable. Es probable que la pérdida oscile o sufra de overshooting (dar saltos demasiado grandes pasando por alto el mínimo de la función de pérdida).

Comportamiento: La precisión sube y baja erráticamente entre épocas adyacentes.


**Pregunta central:** ¿por qué una red puede clasificar ejemplos con buena precisión y aun así chocar?

Una precisión alta solo significa que la red acierta casi todas sus respuestas individuales en papel, pero el juego ocurre en tiempo real y perdona muy pocos errores. Esto pasa por tres razones principales:

Un solo error te mata: La precisión mide el promedio de aciertos. Si la red acierta el 98% del tiempo pero se equivoca en el único fotograma clave donde debía saltar, la partida termina de inmediato. De nada sirve haber acertado los 100 fotogramas anteriores donde no había peligro.

El salto tarda en ejecutarse: La red evalúa si saltar instante por instante, pero la simulación tiene gravedad y física. Si la red decide saltar apenas un fotograma tarde, su respuesta teórica puede parecer correcta en ese instante, pero físicamente el personaje ya no alcanzará la altura suficiente a tiempo para esquivar el obstáculo.

No aprende física, solo imita una regla: La red no comprende cómo funciona la gravedad de Pygame ni la velocidad de caída. Simplemente memorizó una regla matemática artificial (saltar cuando el tiempo estimado esté entre 7 y 13 fotogramas). Si esa regla falla en casos extremos de velocidad, la red seguirá la regla a la perfección pero terminará chocando.

