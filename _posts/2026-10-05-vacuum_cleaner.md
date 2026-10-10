---
title: "Práctica 1: Aspiradora básica"
date: 2026-10-05 01:00:00 +0200
image:
  path: /assets/img/portada_aspiradora.png
  alt: Mapa de cobertura al terminar la prueba
tags: [python, autómata, robótica]
description: "Aspiradora con navegación pseudoaleatoria mediante una máquina de estados"
pin: false
---

## Objetivo
La aspiradora debe cubrir la mayor parte posible de la casa moviéndose de forma pseudoaleatoria, es decir, sin plan ni mapa. A base de moverse "al azar" durante suficiente tiempo, acaba pasando por casi todas partes.

El robot se controla mediante dos velocidades:
- **V (velocidad lineal):** hace avanzar o retroceder al robot
- **W (velocidad angular):** hace girar al robot

> No se permite usar `sleep`, porque bloquea el bucle y el robot deja de reaccionar
{: .prompt-warning }

## Diseño de la FSM
El comportamiento de la aspiradora se organiza en cuatro estados:

![Máquina de estados](/assets/img/maquina_estados.png)
_Máquina de estados de la aspiradora_

### Pseudocódigo

```plaintext
estado = ESPIRAL

Repetir para siempre:
    Si estado == espiral
        W fija y V que aumenta con el tiempo
        si hay algo delante, pasa al siguiente estado: RETROCEDER
        si la espiral ya es muy grande, pasa al siguiente estado: AVANZAR

    También si estado == avanzar
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
Para girar, guardo hacia dónde mira el robot (su yaw) y lo dejo girar hasta que la diferencia llega a un ángulo aleatorio entre 60º y 170º. Tuve que corregir el salto del yaw de 180º a -180º, porque si no, la resta fallaba al pasar por ese punto.

Como no se podía usar `sleep`, guardo la hora cada vez que el robot cambia de estado y en cada vuelta del bucle miro cuánto tiempo lleva en él. Así el robot nunca deja de mirar el láser, y lo uso para que retroceda medio segundo antes de girar.

## Problemas y ajustes
<-- Mencionar lo de podman -->

Al probarlo, la simulación iba muy lenta, así que subí la velocidad de avance a 0,6 m/s y la de giro a 1,5 rad/s. Para que siguiera frenando a tiempo, también aumenté la distancia de seguridad a 45 cm.

Al principio solo miraba los rayos del 70 al 110, y el robot acababa rozando los muebles con los lados. Ampliarlo al 45-135 lo mejoró.

Por último, la espiral se notó bastante, sin ella el robot llegó a un 14,83 % y con ella a un 24,81 %, además de llegar a otra habitación. También probé a que girara hacia un lado aleatorio, pero no noté mejora y lo dejé como estaba.

## Resultados

{% include embed/youtube.html id='Zf_p245PtUw' %}

Esta práctica corresponde al ejercicio [Basic Vacuum Cleaner](https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/vacuum_cleaner) de Robotics Academy.