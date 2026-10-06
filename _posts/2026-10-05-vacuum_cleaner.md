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

**Detección de obstáculos.** Como el bumper está desactivado, uso el láser. De sus 180 medidas, miro las que van de la 45 a la 135 (la 90 es la del frente), para ver también lo que hay un poco a los lados. Si alguna marca menos de 45 cm, el robot deja de avanzar.

**Medir el tiempo sin sleep.** Al cambiar de estado apunto la hora, y en cada vuelta del bucle miro cuánto tiempo lleva en él. Así el robot nunca se bloquea y sigue atento al láser. Lo uso para retroceder medio segundo a 0,2 m/s, unos 10 cm, lo justo para separarse de la pared, ya que va a ciegas hacia atrás.

**Giro con la orientación.** Antes de girar, guardo hacia dónde mira el robot (su yaw) y elijo un ángulo aleatorio entre 60º y 170º. Gira hasta que la diferencia con la orientación inicial llega a ese ángulo. Como el yaw salta de 180º a -180º, corrijo la resta para que no falle en ese punto. Al ser aleatorio, cada vez sale en una dirección distinta y cubre más casa.

**Espiral.** Al arrancar, el robot gira con W fija mientras sube poco a poco V. Como el radio es:

$$
r = \frac{V}{W}
$$

las vueltas se van abriendo. Sale de la espiral si encuentra un obstáculo o cuando V llega a la velocidad de avance normal.

## Problemas y ajustes

**Entorno de ejecución.** En los ordenadores del laboratorio no tenía permisos para usar Docker, así que ejecuté el simulador con Podman, guardando las imágenes en el disco local porque la carpeta personal está en un servidor de red.

**Velocidad de la simulación.** 


<!-- ## Resultado-->

<!-- Vídeo final -->
<!-- Captura con el porcentaje de cobertura -->

Esta práctica corresponde al ejercicio [Basic Vacuum Cleaner](https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/vacuum_cleaner) de Robotics Academy.