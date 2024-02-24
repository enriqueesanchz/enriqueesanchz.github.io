---
title: WorkoutAI - Identificación del ejercicio físico
date: 2024-02-24 15:00:00 +0200
categories: [AI]
tags: [ai, workout, fitness]
---
<style type="text/css">
 .post-content { text-align: justify; }
</style>
Soy una persona a la que le encanta el ejercicio físico, deporte, etc. y voy al gimnasio diariamente. En los inicios del entrenamiento físico consigues un gran cambio, progresas semana a semana y te sientes muy realizado, pero la cosa es diferente cuando pasas ese periodo, llevas más tiempo y te estancas.

Empiezas a creer que no hay mejoras, que la forma de entrenar no es la correcta... Es aquí cuando llega la necesidad de controlar y apuntar el ejercicio que realizas.

| Peso | Series | Repeticiones | RIR  | Fecha             |
| :--- | :----- | ------------ | ---- | ----------------- |
| 100  | 3      | 10           | 0    | 23 noviembre 2023 |
| 105  | 3      | 6            | 0    | 30 noviembre 2023 |
| 100  | 3      | 6            | 1    | 13 diciembre 2023 |
| 105  | 3      | 8            | 0    | 20 diciembre 2023 |

Desde septiembre de 2023 llevo un control sobre el peso, número de repeticiones, series y RIR (repeticiones en reserva) de cada ejercicio de mi entrenamiento diario. Este control se vuelve repetitivo día tras día y, como buen ingeniero, odio las tareas innecesarias y repetitivas, buscando ante todo, **automatizar**.

## La idea

Mi amigo y compañero del Máster, [Manuel Jerez](https://www.linkedin.com/in/manuel-jerez-gonzalez/), y yo nos pusimos manos a la obra. Queríamos crear una app que automatice el apuntado de estas medidas.

Para ello, hay que resolver una serie de problemas:

- Qué ejercicio se está realizando
- Qué peso se está levantando
- Número de series
- Número de repeticiones
- RIR (relacionado con el esfuerzo)

En este artículo trataré el **reconocimiento del ejercicio que se está realizando y el número de repeticiones**.

## Datos

Para conseguir el objetivo, necesitamos unos datos de entrada. Estos serán los proporcionados por un acelerómetro y giroscopio que el sujeto portará atado a la muñeca (como un reloj inteligente), en este caso, prototipado con una Raspberry Pi y un Sense Hat.

![Prototipo de wereable con acelerómetro y giroscopio](/assets/img/workout-wrist-pi.png)

De esta manera, los datos de entrada serán las aceleraciones y velocidades angulares en los 3 ejes.

![Datos de entrada - acceleration](/assets/img/workout-acc.png)

![Datos de entrada - gyro](/assets/img/workout-gyro.png)

### Toma de datos

Se han tomado datos para 10 tipos de ejercicio diferentes, que han sido escogidos de manera que cada uno se parezca a algunos de los demás, para  demostrar que es capaz de diferenciar entre patrones de movimiento similares. Concretamente:

- Press militar
- Press inclinado
- Press sentado
- Bíceps martillo
- Bíceps unilateral polea
- Tríceps polea
- Tríceps unilateral polea
- Remo bajo
- Jalón unilateral
- Jalón al pecho

![Gif de ejercicios](/assets/img/workout.gif)

## Primera iteración

En un principio intentamos reconocer el tipo de ejercicio dividiendo los datos en trozos de 40 muestras, sin tener en cuenta si la repetición estaba empezando, finalizando, centrada, etc.

A partir de estos datos en trozos, entrenamos varias redes neuronales, llegando algunas a ser bastante complejas. Los resultados no fueron buenos, se obtuvo un máximo de 65% de accuracy.

Tras esto, nos centramos en realizar un buen preprocesado de datos para la segunda iteración.

## Segunda iteración

Para esta iteración nos centramos más en tratar los datos de una manera más inteligente. Para ello realizamos una división de los datos en repeticiones.

### Número de repeticiones

Para identificar el número de repeticiones realizadas vamos a utilizar un algoritmo determinista, basado en el cálculo de un umbral. Cuando los valores de aceleración/velocidad sean mayores que este umbral, identificamos una repetición.

![Datos sin dividir](/assets/img/workout-repeticiones-umbral.png)

Para escoger el umbral de manera automática se ha realizado el siguiente proceso:

- Se calcula la desviación estándar de los datos en cada eje: El eje donde esta sea mayor, tendrá una mayor variación durante el ejercicio físico, por lo que será más fácil distinguir las repeticiones fijándonos en este eje.
- Calculamos el percentil 90 de los datos de dicho eje: De esta manera obtenemos un valor mayor que el 90% de los datos, asumiendo que el número de datos de pico de repetición será el 10% restante.

Haciendo uso de este método y eliminando los datos que no consideramos repetición nos queda lo siguiente (zoom):

![Repeticiones](/assets/img/workout-repeticiones-separadas.png)

Si colocamos una repetición sobre otra podemos ver el parecido que tienen todas según el tipo de ejercicio del que se trate.

![Repeticiones - una sobre otra](/assets/img/workout-repeticiones-encima.png)

### Identificación del ejercicio

Estamos ante un problema de clasificación. Para resolverlo vamos a usar una Red Neuronal Convolucional 1D, con una capa densa y una capa final softmax:

![CNN 1D](/assets/img/workout-net.png)

Se ha elegido este tipo de red porque:

- Es simple
- Ocupa pocos recursos, por lo que se podría usar incluso en sistemas empotrados: **87 KB**, es decir, mucho menos que este post de mi blog o cualquier imagen de las que enlaza.
- Es suficiente para obtener buenos resultados (para el conjunto de 10 tipos de ejercicios)

Se obtienen los siguientes resultados para los datos de test, fallando tan solo en 1 repetición entre dos ejercicios muy parecidos en cuanto a movimiento como son el press inclinado y el press militar.

![Matriz de confusión](/assets/img/workout-matrix.png)

## Conclusiones

Se ha conseguido identificar los diferentes ejercicios con un 99% de accuracy en test. Esto no es lo más relevante, puesto que, existen factores no estudiados como qué pasaría con más de 10 clases de ejercicios, qué pasaría al introducir outliers, etc.

La conclusión más importante que obtenemos es **la importancia del dato**. En la primera iteración, por mucho que añadiesemos complejidad a la red, no llegábamos a unos buenos resultados. Sin embargo, en la segunda iteración, tratando cuidadosamente los datos, se consigue una accuracy muy elevada, incluso con redes mucho más simples que las anteriormente evaluadas.

## Leer más

En un futuro escribiré sobre el resto de problemas a resolver de la aplicación, enlazándolos abajo:

- Identificación del RIR
- Número de series
- Peso

Tanto los datos como el código (Jupyter Notebooks) están disponibles en mi [Repositorio de Github](https://github.com/enriqueesanchz/workoutAI).
