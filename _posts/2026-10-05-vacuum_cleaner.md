---
title: "Práctica 1: Aspiradora básica"
date: 2026-10-05 01:00:00 +0200
tags: [python, autómata, robotica]
description: "Aspiradora con navegación pseudoaleatoria mediante una máquina de estados"
pin: true
---

## Objetivo
El objetivo es que la aspiradora cubra la mayor parte posible de la casa moviéndose de forma pseudoaleatoria, es decir, sin plan ni mapa. A base de moverse "al azar" durante suficiente tiempo, acaba pasando por casi todas partes. Para ello, la práctica pide implementar un autómata con al menos 3 estados (avanzando, retrocediendo y girando). Además, he añadido un cuarto estado opcional, la espiral, que mejora la cobertura al inicio.

El robot se controla mediante dos velocidades:
- **V (velocidad lineal):** hace avanzar o retroceder al robot
- **W (velocidad angular):** hace girar al robot

> No se permite usar `sleep`, porque bloquea el bucle y el robot deja de reaccionar
{: .prompt-warning }

## Diseño de la FSM
El comportamiento de la aspiradora se organiza en tres estados:

![Máquina de estados](/assets/img/maquina_estados.png)
_Máquina de estados de la aspiradora_

### Pseudocódigo

```plaintext
estado = ESPIRAL

Repetir para siempre:
    Si estado == espiral
        W fija y V que aumenta con el tiempo
        si hay algo delante, pasa al siguiente estado: RETROCEDER
        si la espiral ya es muy grande: pasa al siguiente estado: AVANZAR

    Si estado == avanzar
        V positiva y W = 0
        termina si hay algo delante que le impide avanzar
        pasa al siguiente estado: RETROCEDER

    También si estado == retroceder
        V negativa y W = 0
        termina cuando pasa un tiempo razonable (medio segundo)
        guarda hacia dónde mira
        elige cuánto girar (un ángulo aleatorio)
        pasa al siguiente estado: GIRAR

    También si estado == girar
        V = 0 y W positiva
        termina cuando ha girado el ángulo aleatorio elegido
        pasa al siguiente estado: AVANZAR
```
{: file='Pseudocódigo' .nolineno}

## Implementación

**Detección de obstáculos.** El bumper de la aspiradora está desactivado, así que para saber si va a chocar uso el láser. El láser me da 180 medidas de
distancia, una por cada grado, y la del medio (la 90) es la que mira justo al frente. En vez de fiarme solo de esa, miro las que van de la 45 a la 135, para ver también lo que tiene un poco a los lados. Si alguna marca menos de 45 cm, el robot entiende que tiene algo delante y deja de avanzar.

Para comprobar que le da tiempo a frenar, calculo cuánto avanza entre dos lecturas del láser. El bucle va a unos 30 Hz, así que:

$$
d = V \cdot \Delta t = 0{,}6 \, \text{m/s} \cdot \tfrac{1}{30} \, \text{s} = 0{,}02 \, \text{m} = 2 \, \text{cm}
$$

Son solo 2 cm frente a los 45 cm de margen, así que siempre detecta el obstáculo a tiempo.

**Medir el tiempo sin sleep.** Como no se puede usar `sleep`, cada vez que el robot cambia de estado apunto la hora. Después, en cada vuelta del bucle miro
cuánto tiempo lleva en ese estado, y cuando ya ha pasado el que quería, cambia al siguiente. Así el robot nunca se queda "dormido" y sigue pendiente del láser todo el rato.

Lo uso para retroceder: medio segundo a 0,2 m/s, que equivale a

$$
d = V \cdot t = 0{,}2 \, \text{m/s} \cdot 0{,}5 \, \text{s} = 0{,}1 \, \text{m} = 10 \, \text{cm}
$$

Lo justo para despegarse de la pared, sin ir demasiado tiempo a ciegas, ya que no tiene sensor por detrás.

**Giro con la orientación.** Antes de girar, apunto hacia dónde está mirando el robot (su yaw) y elijo un ángulo aleatorio entre 60° y 170°. Luego lo dejo
girar hasta que la diferencia entre hacia dónde mira ahora y hacia dónde miraba al principio llega a ese ángulo. Hay un detalle: el yaw salta de 180° a -180°, así que tuve que corregir la resta para que un giro pequeño no pareciera uno enorme. Que el ángulo sea aleatorio hace que cada vez salga en una dirección distinta, y así acaba pasando por más sitios de la casa.

Como la orientación se mide en radianes, el rango de giro queda:

$$
60° \cdot \frac{\pi}{180} \approx 1{,}05 \, \text{rad}, \qquad 170° \cdot \frac{\pi}{180} \approx 2{,}97 \, \text{rad}
$$

El máximo tiene que quedarse por debajo de 180° (π rad): con la corrección del salto, la diferencia nunca pasa de ese valor, así que un objetivo mayor no se alcanzaría nunca y el robot giraría sin parar.

**Espiral.** Al arrancar, el robot gira con una velocidad angular (W) fija mientras va subiendo poco a poco la velocidad lineal (V). Cuanto más rápido
avanza mientras gira, más grandes son las vueltas, porque el radio cumple:

$$
r = \frac{V}{W}
$$

Así dibuja una espiral que se va abriendo y limpia bien la zona donde empieza. Deja la espiral cuando encuentra un obstáculo o cuando la velocidad llega a la de avance normal, que es cuando las vueltas ya son demasiado grandes:

$$
r_{max} = \frac{0{,}6 \, \text{m/s}}{1{,}0 \, \text{rad/s}} = 0{,}6 \, \text{m}
$$

Partiendo de 0,05 m/s y subiendo 0,015 m/s cada segundo, la espiral dura unos

$$
t = \frac{0{,}6 - 0{,}05}{0{,}015} \approx 37 \, \text{s}
$$

## Problemas y ajustes

**Entorno de ejecución.** En los ordenadores del laboratorio no tenía permisos para usar Docker, así que ejecuté el simulador con Podman, guardando las imágenes en el disco local porque la carpeta personal está en un servidor de red.

**Velocidad de la simulación.** 


<!-- ## Resultado-->

<!-- Vídeo final -->
<!-- Captura con el porcentaje de cobertura -->

Esta práctica corresponde al ejercicio [Basic Vacuum Cleaner](https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/vacuum_cleaner) de Robotics Academy.