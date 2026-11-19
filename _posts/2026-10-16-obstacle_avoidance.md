---
title: "Práctica 3: Local navigation with VFF"
date: 2026-10-16 01:00:00 +0200
tags: [navegación local, vff, python]
description: "Fórmula 1 que esquiva obstáculos y alcanza subobjetivos con VFF"
pin: false
math: true
---

## Objetivo
Programar un Fórmula 1 que recorra el circuito pasando por una serie de **subobjetivos** sin chocar con los obstáculos. No tiene mapa, solo usa el **láser** y su **posición**.

Lo que se pedía:
- Que no choque.
- Que alcance todos los subobjetivos.
- Que no oscile y que el movimiento sea fluido.

## Implementación
He usado **navegación local**. El coche decide solo con lo que ve el láser en cada momento. Es rápida y reacciona bien ante imprevistos, pero hasta que no ve un obstáculo, no lo esquiva.

En concreto, uso **VFF** (campo de fuerzas virtual). En cada iteración calculo una fuerza **atractiva** hacia el subobjetivo y una **repulsiva** que aleja al coche de los obstáculos. La suma de las dos decide hacia dónde gira.

### Lectura del láser
El láser da 180 distancias, una por grado, de derecha a izquierda. A cada lectura $ i $ le asigno su ángulo:

$$
\theta_i = i - 90^\circ
$$

Así la lectura 0 queda a la derecha, la 90 justo delante y la 179 a la izquierda. Con la distancia y el ángulo paso cada lectura a un punto $ (x, y) $ respecto al coche, con $ x $ hacia delante e $ y $ hacia la izquierda. Esos puntos son los obstáculos que uso para la fuerza repulsiva.

### Sistema de referencia del coche

### Fuerza atractiva

### Fuerza repulsiva

### Fuerza resultante y control

### Subobjetivos

## Vídeo final

Esta práctica corresponde al ejercicio [Obstacle Avoidance](https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/obstacle_avoidance) de Robotics Academy.
