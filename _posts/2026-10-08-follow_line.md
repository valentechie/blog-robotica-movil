---
title: "Práctica 2: Visual follow Line"
date: 2026-10-08 01:00:00 +0200
image:
  path: /assets/img/portada_followline.png
  alt: Fórmula 1 en la simulación
tags: [opencv, pid, control]
description: "Control Visual con PID de un Fórmula1"
pin: true
math: true
---

## Objetivo
El objetivo de esta práctica es programar un Fórmula 1 para que siga la línea roja pintada en el centro del circuito, usando únicamente la información que obtiene de su cámara. El control debe ser **reactivo**, en cada iteración se decide qué hacer a partir de lo que se ve en ese instante, sin mapa ni trayectoria calculada de antemano.

Más allá de completar la vuelta, el controlador debe cumplir cuatro condiciones, no oscilar, ser rápido, funcionar en circuitos distintos sin tocar el código y recuperar la línea si llega a perderla.

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

### Holonómico y no holonómico
 
Un robot que se desplaza por un plano tiene tres grados de libertad, posición $$ x $$, posición $$ y $$ y orientación $$ \theta $$. Un robot **holonómico** los controla de forma independiente y puede desplazarse en cualquier dirección en cualquier instante. Uno **no holonómico** está sujeto a restricciones cinemáticas y necesita maniobrar, como un coche real, que no puede desplazarse lateralmente hacia un hueco de aparcamiento.
 
Esto limita el control de dos formas. El vehículo no holonómico no alcanza cualquier velocidad angular que se le pida, si el giro es más cerrado que su radio mínimo no lo ejecuta, y su dirección tarda en responder, de modo que la corrección llega con retardo. Por eso se implementan dos versiones del controlador, con la misma estructura pero distinto ajuste de ganancias y de velocidad.
 
## Control
 
### Controlador P
 
La primera versión implementa solo el término proporcional, con velocidad lineal constante y reducida. El objetivo no es ir rápido sino comprobar que la cadena completa funciona, que la línea se detecta, que el error tiene el signo correcto y que el coche reacciona en la dirección esperada.
 
```plaintext
mientras sea verdad
    imagen = capturar de la camara
    mascara = filtrar rojo en HSV
    cx = centroide de la mascara
    error = cx - centro_imagen
 
    V = constante baja
    W = Kp * error
 
    aplicar V y W
```
 
Conviene verificar el signo del resultado. Una velocidad angular positiva hace girar al coche hacia la izquierda, mientras que un error positivo significa que la línea está a la derecha, por lo que puede ser necesario invertir el signo.
 

_Coche oscilando con control P_
 
### Controlador PD
 
El control proporcional puro oscila porque corrige en función de dónde está la línea pero no de hacia dónde se está moviendo. Cuando el coche se acerca al centro con mucha velocidad angular el término proporcional ya casi no corrige, pero la inercia del giro lo lleva a pasarse al otro lado, y el ciclo se repite.
 
El término derivativo incorpora esa tendencia. Al depender de la diferencia entre el error actual y el anterior, su signo es contrario al del movimiento cuando el coche se aproxima al centro, de modo que frena el giro antes de que se pase. Requiere guardar el error de la iteración previa.
 
```plaintext
error_prev = 0        # fuera del bucle
 
mientras verdadero
    ...
    derivada = error - error_prev
    W = Kp * error + Kd * derivada
 
    error_prev = error    # al final de cada iteracion
```
 
Además de eliminar el zigzagueo permite subir la velocidad, ya que sin él cualquier aumento amplifica las oscilaciones hasta desestabilizar el coche.
 
### Término integral
 
El término integral corrige desviaciones pequeñas y persistentes, como circular de forma constante ligeramente desplazado respecto a la línea. Es imprescindible limitar su acumulación, lo que se conoce como **anti-windup**, porque de lo contrario la suma crece indefinidamente. En una curva larga alcanza un valor muy alto y, al volver a la recta, el término sigue ordenando girar durante un tiempo.
 
```plaintext
suma_error = suma_error + error
suma_error = limitar(suma_error, -MAX, +MAX)
W = Kp * error + Ki * suma_error + Kd * derivada
```
 
En un seguidor de línea el error no suele tener un sesgo permanente porque el trazado cambia continuamente de dirección, así que un PD bien ajustado suele ser suficiente.

## Recuperación de la línea

El coche puede perder la línea si llega a una curva demasiado rápido o si se desvía lo suficiente como para que la línea salga del campo de visión de la cámara. En ese caso el controlador no dispone de error que calcular y queda inservible, por lo que el comportamiento se organiza como una máquina de estados con dos estados:

![FSM](/assets/img/recu_linea.png)
_FSM de recuperación de la línea_

- **Detección de la pérdida:** se considera que la línea no es visible cuando el área de la máscara ($$ m_{00} $$) queda por debajo de un umbral mínimo. Comparar contra un umbral, en lugar de contra cero, evita que unos pocos píxeles sueltos de ruido se interpreten como línea, y de paso impide la división por cero en $$ \eqref{eq:centroide} $$.
- **Dirección de búsqueda:** se guarda el **signo del último error** conocido, que indica por qué lado se escapó la línea, y se gira hacia ese lado. Girar en la dirección equivocada alejaría definitivamente el coche del trazado.
- **Parámetros de búsqueda:** velocidad lineal baja y velocidad angular elevada, para maniobrar sin alejarse más del trazado.
- **Retorno:** en cuanto el área de la máscara vuelve a superar el umbral, se recupera el estado SEGUIR.

```plaintext
si area_mascara > UMBRAL:
    estado <- SEGUIR
    calcular error y aplicar PID
    ultimo_signo <- signo(error)
sino:
    estado <- BUSCAR
    V <- baja
    W <- ultimo_signo * W_busqueda
```


Esta práctica corresponde al ejercicio [Visual follow Line](https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/) de Robotics Academy.