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

**Detección de obstáculos**
- El bumper está desactivado, así que uso el láser
- Miro las medidas de la 45 a la 135 (la 90 es el frente) para ver también los lados
- Si alguna marca menos de 45 cm, hay obstáculo

**Tiempo sin sleep**
- Al cambiar de estado guardo la hora y en cada vuelta miro cuánto lleva
- El robot nunca se bloquea y sigue atento al láser
- Lo uso para retroceder 0,5 s a 0,2 m/s (unos 10 cm)

**Giro con la orientación**
- Guardo el yaw al empezar y elijo un ángulo aleatorio entre 60º y 170º
- Gira hasta que la diferencia con el yaw inicial llega a ese ángulo
- Corrijo el salto del yaw de 180º a -180º

**Espiral**
- W fija y V creciente: como `r = V / W`, las vueltas se abren.
- Termina si hay un obstáculo o cuando V llega a la velocidad de avance.

## Problemas y ajustes

**Giro por tiempo o por ángulo.** La página del ejercicio recomienda girar durante un tiempo aleatorio y usar `sleep` para esperar. Como el enunciado de la asignatura no permite `sleep` y sí deja usar la orientación, decidí medir el giro con el yaw. Así no hace falta girar un ángulo exacto: basta con girar aproximadamente uno aleatorio, que es lo que necesita la navegación pseudoaleatoria.

**Tiempo real y tiempo simulado.** Los tiempos del código se miden con el reloj del ordenador, no con el del simulador. Como la simulación iba a un 30 % del tiempo real, el robot retrocedía menos distancia de la calculada y la espiral se abría más despacio. Aun así el comportamiento fue correcto, pero en un ordenador con GPU estos valores podrían necesitar un ajuste.

## Resultados

### Vídeo final
{% include embed/youtube.html id='Zf_p245PtUw' %}

### Captura con el porcentaje de cobertura
![Resultado final](/assets/img/resultado.png)
_Cobertura final: 36,61% tras 11,03 minutos_


Esta práctica corresponde al ejercicio [Basic Vacuum Cleaner](https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/vacuum_cleaner) de Robotics Academy.