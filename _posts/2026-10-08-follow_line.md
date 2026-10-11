---
title: "Práctica 2: Visual follow Line"
date: 2026-10-08 01:00:00 +0200
tags: [python, PID, robótica]
description: "Control Visual con PID de un Fórmula1"
pin: true
---

## Objetivo
Un Fórmula 1 simulado que tiene que seguir la línea roja pintada en el centro del circuito, usando solo lo que ve su cámara. La práctica pide hacerlo con un controlador PID reactivo.

## Sistema de control
![Esquema del sistema](/assets/img/sistema_control.png)

## Diagrama de estados (Para el while)

1. Get camera image
2. Detect the red line
3. Finds position
4. Compute error
5. Run PID Controller
6. Send velocity commands
7. Repetir

## Pseudocodigo
```
Mientras coche encendido:
    mostrar la imagen de la camara
    

```

## PID Controller


## Fundamentos de diseño
<-- dos codigos: uno para el holonomico y otro para el no holonomico-->


Esta práctica corresponde al ejercicio [Visual follow Line](https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/) de Robotics Academy.