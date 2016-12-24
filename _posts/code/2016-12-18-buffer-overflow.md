---
layout: post
title: Buffer Overflow
excerpt: Introducción a la explotación del buffer overflow
date: 2012-12-18
categories: code
comments: true
share: true
tags: [security, buffer overflow, exploit]
---
En este artículo intento explicar con ejemplos cómo funciona un desbordamiento de buffer o _buffer overflow_. Se trata de un tipo de vulnerabilidad que se explota escribiendo más allá del final de un buffer, de tal forma que se modifique la dirección de retorno y se ejecutando un código arbitrario.

## Organización de la memoria en los procesos

Antes de nada es conveniente tener una cierta idea de cómo se organiza un proceso en memoria, qué es el stack, qué ocurre cuando se llama a una función y cuando se vuelve...

![Process Memory Organization]({{ site.url }}/images/process_mem_organization.png)

El stack es una estructura LIFO en la que se almacenan los parámetros de llamada funciones, direcciones de retorno y variables locales a la función en ejecución. Crece "hacia abajo", es decir, hacia direcciones bajas de memoria.

El _heap_, al contrario que el _stack_, crece hacia arriba. En él se almecena memoria reservada dinámicamente (como mediante `malloc`).

Los segmentos _bss_ y _data_ almacenan variables (globales y estáticas) no inicializadas e inicializadas respectivamente.

Finalmente, el segmento _text_ contiene el código ejecutable.

## Un programa vulnerable
