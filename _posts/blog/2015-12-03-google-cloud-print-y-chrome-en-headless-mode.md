---
layout: post
title:  "Google Cloud Print y Chrome en \"headless mode\""
excerpt: Arrancar Google Chrome en Xvfb
date:   2015-12-03
categories: blog
comments: true
share: true
tags: [google, chrome, print]
---

La verdad es que Google Cloud Print es cómodo para ciertos usos (por ejemplo, imprimir desde un Android). Pero tiene un pequeño inconveniente: requiere que el servidor de impresora tenga Google Chrome arrancado, lo que no necesariamente ocurre.

Bueno, pues hay una solución muy fácil mediante Xvfb (virtual framebuffer), un servidor “gráfico” que no muestra nada por pantalla. Así:

```
xvfb-run --server-args='-screen 0, 1024x768x24' google-chrome \
-start-maximized http://debian.org > /dev/null &
```
