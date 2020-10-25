---
published: false
title: 'Autorización con roles en NestJS'
cover_image:
description:
tags: nestjs
series: NestJS
canonical_url:
---

# Resumen

Vamos a ver cómo 

# Entorno

- Slimbook Prox15 32 Gb RAM i7
- SlimbookOS (Ubuntu 20.04)
- NestJS 7.0.0
- Visual Studio Code

# Introducción

La autenticación con token de JWT se ha convertido en un estándar de facto a la hora de proteger los endpoints más sensibles de nuestra aplicación: aquellos que crean, modifican o eliminan recursos de nuestro servidor y otros de consulta que devuelven información sensible que solo debería estar accesible por el usuario en concreto.

De hecho aquí vamos a ver como enviar el id del usuario a través de token, algo mucho más seguro que hacerlo por parámetro en la URL. Imaginad un endpoint que devuelve datos judiciales, y que simplemente cambiando el id por parámetro y volviendo hacer la llamada, devuelve los datos judiciales, que obviamente tienen que ser confidenciales, de cualquier persona que no conoces de nada, pues eso ha pasado....

# Vamos al lío

