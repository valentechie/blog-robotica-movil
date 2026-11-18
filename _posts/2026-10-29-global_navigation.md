---
title: "Práctica 4: Navegación global con algoritmo GPP"
date: 2026-10-29 01:00:00 +0200
tags: [global_navigation, gpp, planning]
description: "Global Navigation using TeleTaxi"
pin: true
math: true
---

## Objetivo
El objetivo de esta práctica es programar un coche autónomo que haga de taxi en una ciudad. El coche conoce su posición gracias a un GPS y dispone de un mapa de la ciudad. Cuando el usuario marca un destino haciendo clic sobre el mapa, el taxi debe **planificar** cómo llegar desde donde está y después conducir hasta allí sin chocar con los edificios.

La planificación se hace con el algoritmo **Gradient Path Planning (GPP)**, y la conducción con un bucle reactivo que fija la velocidad lineal $ V $ y la velocidad angular $ W $. Al llegar, el taxi espera a que el usuario marque un nuevo destino.

## Implementación

### Exploración del mapa
El mapa es una matriz de $ 400 \times 400 $ (cada celda mide $ 1{,}25 $ m) con dos valores:

| Valor | Significado |
|:-----:|:------------|
| 255 | Calle |
| 0 | Edificio |

Lo comprobé mirando la celda del taxi, que empieza en una calle.

Al pasar un punto del mundo a celda se obtienen dos números $ (a, b) $, pero el mapa hay que leerlo **al revés**, como `mapa[b][a]`. Lo comprobé haciendo clic en varias calles, solo en ese orden salía siempre 255. Para no tener que recordarlo, lo metí en una función.
