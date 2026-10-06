---
title: "Práctica 1: Aspiradora básica"
date: 2026-10-05 01:00:00 +0200
tags: [python, autómata, robotica]
description: "Aspiradora con navegación pseudoaleatoria mediante una máquina de estados"
pin: true
---

## Objetivo
La aspiradora debe cubrir la mayor parte posible de la casa moviéndose de forma pseudoaleatoria, es decir, sin plan ni mapa. A base de moverse "al azar" durante suficiente tiempo, acaba pasando por casi todas partes. Para ello, la práctica pide implementar un autómata con al menos 3 estados (avanzando, retrocediendo y girando). Además, he añadido un cuarto estado opcional, la espiral, que mejora la cobertura al inicio.

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

## Resultados

### Vídeo final
{% include embed/youtube.html id='Zf_p245PtUw' %}
_Ejecución de la aspiradora_

### Captura con el porcentaje de cobertura
En otra de las pruebas, tras 11min 3s, el robot llegó a un 36,61 % de cobertura:

![Resultado de otra prueba](/assets/img/resultado.png)
_Mapa de cobertura al terminar la prueba_


Esta práctica corresponde al ejercicio [Basic Vacuum Cleaner](https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/vacuum_cleaner) de Robotics Academy.