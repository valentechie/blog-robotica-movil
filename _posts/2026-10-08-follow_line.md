---
title: "Práctica 2: Visual follow Line"
date: 2026-10-08 01:00:00 +0200
tags: [python, PID, robótica]
description: "Control Visual con PID de un Fórmula1"
pin: true
math: true
---

## Objetivo
Un Fórmula 1 simulado que tiene que seguir la línea roja pintada en el centro del circuito, usando solo lo que ve su cámara. La práctica pide hacerlo con un controlador PID reactivo.

## Sistema de control
![Esquema del sistema](/assets/img/sistema_control.png)

## Definiciones usadas

### Setpoint
Es la posición en la que queremos estar. En esta práctica es el **centro horizontal de la imagen**: queremos que la línea roja quede justo delante del coche.

### Error
Es la diferencia entre la posición deseada y la posición actual, medida en píxeles. En cada iteración $$ k $$ del bucle:

$$
\begin{equation}
  e_k = x_{\text{línea}} - x_{\text{centro}}
  \label{eq:error}
\end{equation}
$$

### PID
El controlador calcula la velocidad angular $$ W $$ a partir del error $$ \eqref{eq:error} $$, combinando tres términos:

- **P (proporcional):** $$ K_p \, e_k $$. La corrección es proporcional al error: cuanto más lejos está la línea, más gira el coche.
- **I (integral):** $$ K_i \sum_{i=0}^{k} e_i $$. Acumula el error en el tiempo para eliminar desviaciones pequeñas que se mantienen.
- **D (derivativo):** $$ K_d \, (e_k - e_{k-1}) $$. Tiene en cuenta lo rápido que cambia el error y amortigua las oscilaciones.

La velocidad angular final es la suma de los tres términos:

$$
\begin{equation}
  W_k = K_p \, e_k + K_i \sum_{i=0}^{k} e_i + K_d \, (e_k - e_{k-1})
  \label{eq:pid}
\end{equation}
$$

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

## Fundamentos de diseño
<-- dos codigos: uno para el holonomico y otro para el no holonomico-->


Esta práctica corresponde al ejercicio [Visual follow Line](https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/) de Robotics Academy.