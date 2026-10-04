---
title: "Práctica 1: Aspiradora de gama baja"
date: 2026-10-05 01:00:00 +0200
tags: [python, autómata, robotica]
description: "Aspiradora con navegación pseudoaleatoria mediante una máquina de estados"
pin: true
mermaid: true
---

# Objetivo
Que la aspiradora cubra la mayor parte posible de la casa moviéndose de forma pseudoaleatoria, es decir, sin plan ni mapa. A base de moverse "al azar" durante suficiente tiempo, acaba pasando por casi todas partes. Para ello nos pide implementar un automata con al menos 3 estados (avanzando, retrocediendo y girando).

Datos que nos dan:
- V: velocidad linear
- W: velocidad angular

### Máquina de estados
Para realizarla sigo el siguiente esquema:

![Máquina de estados](/assets/img/maquina_estados.png)

```txt
Repetir para siempre:

    Si estado == avanzar
        V positiva y W = 0
        termina si hay algo delante que le impide avanzar
        pasa al siguiente estado: RETROCEDER

    También si estado == retroceder
        V negativa y W = 0
        termina cuando pasa un tiempo razonable (medio seg)
        guarda hacia donde mira
        pasa al siguiente estado: GIRAR

    También si estado == girar
        V = 0 y W positiva
        termina cuando ha girado el ángulo aleatorio elegido
        pasa al siguiente estado: AVANZAR
```
