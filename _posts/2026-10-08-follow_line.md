---
title: "Práctica 2: Visual follow Line"
date: 2026-10-08 01:00:00 +0200
tags: [opencv, pid, control]
description: "Control Visual con PID de un Fórmula1"
pin: true
math: true
---

## Objetivo
El objetivo de esta práctica es programar un Fórmula 1 para que siga la línea roja pintada en el centro del circuito, usando únicamente la información que obtiene de su cámara. El control debe ser **reactivo**, en cada iteración se decide qué hacer a partir de lo que se ve en ese instante, sin mapa ni trayectoria calculada de antemano.

Más allá de completar la vuelta, el controlador debe cumplir cuatro condiciones, no oscilar, ser rápido, funcionar en circuitos distintos sin tocar el código y recuperar la línea si llega a perderla. A esto se suma que el ejercicio se resuelve dos veces, una para el coche holonómico y otra para el no holonómico, porque la forma de moverse de cada uno condiciona el control.

## Fundamentos teóricos

### Sistema de control

Un **sistema de control** es aquel que gobierna el comportamiento de otro sistema para conseguir un resultado deseado. Se distinguen dos tipos:

- **Lazo abierto:** la acción de control es independiente de la salida del sistema. No hay corrección posible: equivale a conducir con los ojos cerrados manteniendo el volante fijo.
- **Lazo cerrado:** la salida influye sobre la entrada mediante **realimentación** (*feedback*). El sistema mide el resultado, lo compara con el valor deseado y corrige la diferencia.

El control de esta práctica es de lazo cerrado. La cámara mide continuamente dónde está la línea, el controlador compara esa posición con la deseada y ajusta el giro; el giro modifica la posición del coche, que la cámara vuelve a medir en la siguiente iteración. El bucle se repite unas cincuenta veces por segundo.

![Esquema del sistema de control](/assets/img/sistema_control.png)
_Bucle de control cerrado implementado en la práctica_

### Definiciones

#### Setpoint (consigna)

Es la posición en la que queremos estar. En esta práctica es el **centro horizontal de la imagen**, buscamos que la línea roja quede justo delante del coche. Si la línea aparece centrada en la imagen, el coche está bien orientado y no hay nada que corregir.

#### Error

Diferencia entre la posición deseada y la posición actual, medida en píxeles. En cada iteración $$ k $$ del bucle de control:

$$
\begin{equation}
  e_k = x_{\text{línea}} - x_{\text{centro}}
  \label{eq:error}
\end{equation}
$$

El **signo** del error indica hacia qué lado se ha desviado el coche y su **magnitud**, cuánto. Ambas informaciones se aprovechan, el signo determina la dirección del giro y también hacia dónde buscar la línea si se pierde, mientras que la magnitud determina la intensidad de la corrección y la velocidad a la que conviene circular.

#### PID

El controlador calcula la velocidad angular $$ W $$ a partir del error $$ \eqref{eq:error} $$, combinando tres términos:

- **P (proporcional):** $$ K_p \, e_k $$. La corrección es proporcional al error actual, cuanto más lejos está la línea, más gira el coche. Si el error es nulo, no corrige. Por sí solo tiende a pasarse de largo, porque no tiene forma de anticipar que el error ya está disminuyendo.
- **I (integral):** $$ K_i \sum_{i=0}^{k} e_i $$. Acumula el error a lo largo del tiempo, eliminando desviaciones pequeñas y persistentes que el término proporcional no llega a corregir porque generan una corrección demasiado débil.
- **D (derivativo):** $$ K_d \, (e_k - e_{k-1}) $$. Depende de la velocidad de variación del error. Actúa como un amortiguador, frena la corrección cuando el error ya está disminuyendo, lo que reduce las oscilaciones.

La velocidad angular resultante es la suma de los tres términos:

$$
\begin{equation}
  W_k = K_p \, e_k + K_i \sum_{i=0}^{k} e_i + K_d \, (e_k - e_{k-1})
  \label{eq:pid}
\end{equation}
$$

Como las imágenes llegan de una en una, trabajamos en **tiempo discreto**, la integral se aproxima mediante una suma acumulada y la derivada, mediante la diferencia entre el error actual y el de la iteración anterior.

### Ajuste de las ganancias

Las constantes $$ K_p $$, $$ K_i $$ y $$ K_d $$ se ajustan experimentalmente. El método de **prueba y error** consiste en partir de $$ K_i = K_d = 0 $$, aumentar $$ K_p $$ hasta que el sistema oscile y añadir después el término derivativo para amortiguar esas oscilaciones.

Conviene tener en cuenta el orden de magnitud de cada variable, el error se mide en píxeles, por lo que alcanza valores de centenares, mientras que la velocidad angular se expresa en radianes por segundo y se mueve en el entorno de la unidad. En consecuencia, $$ K_p $$ toma valores muy pequeños.

Los dos comportamientos defectuosos típicos son:

| Síntoma | Descripción | Causa habitual |
|:--------|:------------|:---------------|
| Oscilaciones inestables | El coche zigzaguea con amplitud creciente hasta salirse | $$ K_p $$ demasiado alto o $$ K_d $$ insuficiente |
| Respuesta lenta | El coche no llega a tiempo a las curvas | $$ K_p $$ demasiado bajo o velocidad excesiva |

### Holonómico vs. no holonómico

Un robot que se desplaza por un plano tiene tres grados de libertad, posición $$ x $$, posición $$ y $$ y orientación $$ \theta $$.

- **Holonómico:** puede controlar los tres grados de libertad de forma independiente, de modo que es capaz de desplazarse en cualquier dirección en cualquier instante.
- **No holonómico:** está sujeto a **restricciones cinemáticas**. No puede moverse directamente en cualquier dirección, sino que necesita maniobrar para alcanzar una posición.

Un coche real es no holonómico, no puede desplazarse lateralmente hacia un hueco de aparcamiento, sino que tiene que maniobrar para entrar en él.



El vehículo no holonómico no alcanza cualquier velocidad angular que se le solicite. Si se le pide un giro más cerrado que su radio mínimo, simplemente no lo ejecuta. Además, la dirección tarda en responder, de forma que la corrección llega con retardo respecto al momento en que se calculó.

Por eso se implementan **dos versiones del controlador**. La estructura es idéntica, pero el ajuste de las ganancias y la gestión de la velocidad difieren.




Esta práctica corresponde al ejercicio [Visual follow Line](https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/) de Robotics Academy.