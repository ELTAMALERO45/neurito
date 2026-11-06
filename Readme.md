1. ¿En qué se diferencian pesos y sesgos? 
Los pesos (W) actúan como ponderadores de importancia que controlan la pendiente y 
orientación de las fronteras de decisión en el espacio de entradas. Los sesgos (b) son 
constantes independientes que permiten desplazar la función de activación horizontalmente, 
otorgándole a la neurona la capacidad de activarse aun cuando todas las entradas tengan 
valor cero. 
2. ¿Por qué el cansancio es 1 energía? 
La red neuronal opera evaluando urgencias o necesidades pasivas (a mayor valor, mayor 
prioridad). La variable energía mide la reserva actual (1.0 lleno, 0.0 agotado). Invertir este 
valor (1 energía) traduce el estado interno en una métrica de cansancio directa: cuanta 
menos energía le queda a Neurito, mayor es la prioridad numérica asignada a la acción de 
dormir. 
3. ¿Qué hace ReLU y qué hace ? 
• ReLU (max(0, z)): Es la función de activación no lineal en la capa oculta. 
Transforma las combinaciones lineales eliminando valores negativos, lo que permite 
a la red aprender fronteras complejas manteniendo un cálculo de gradientes 
eficiente. 
• Softmax: Se ubica en la capa de salida. Convierte el vector de puntuaciones 
continuas (logits) en una distribución de probabilidad normalizada cuyos 
componentes son todos positivos y suman exactamente 1.0 (100) 

4. ¿Por qué una precisión alta puede ser engañosa? 
Una precisión global elevada (ej. 98) puede enmascarar deficiencias en el modelo debido a: 
1. Casos límite con baja certeza: En situaciones con necesidades casi idénticas, el     
modelo puede acertar la clase objetivo por diferencias insignificantes (33.4 vs 33.3), 
registrando una predicción "correcta" a pesar de tener una certidumbre deficiente. 
2. Desbalance de datos o regiones no exploradas: Si la muestra sintética no 
distribuye equitativamente las situaciones extremas, la métrica global de precisión 
ocultará fallas graves en escenarios poco frecuentes pero críticos para el sistema.