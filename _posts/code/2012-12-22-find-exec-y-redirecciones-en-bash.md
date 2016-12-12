---
layout: post
title:  Find, exec y redirecciones en bash
excerpt: Cuando find no nos hace ni puto caso
date:   2012-12-22
categories: code
comments: true
share: true
tags: [shell, bash]
---

Es habitual la necesidad de lanzar comandos de este tipo: buscar una serie de ficheros y, una vez encontrados, escribir en cada uno de ellos determinada cadena. Un ejemplo práctico sería buscar los ficheros scheduler para dispositivos de disco y escribir en ellos el nombre del *elevator* deseado.

Lo que pide el cuerpo es hacer algo así:

{% highlight shell %}
find /sys/block/sd*/queue -maxdepth 1 -name scheduler \
  -exec echo "noop" > {} \;
{% endhighlight  %}

Algo no cuadra, ¿verdad? Esa redirección no tiene buena pinta. De hecho, lo más probable es que terminemos con un fichero de nombre “{}” en el directorio actual con tantas líneas “noop” como dispositivos encontrados

Hay varias alternativas pero la más sencilla es la siguiente:

{% highlight shell %}
find /sys/block/sd*/queue -maxdepth 1 -name scheduler \
  -exec sh -c 'echo "noop" > {}' \;  
{% endhighlight  %}

La redirección se interpretará correctamente y obtendremos el resultado deseado.
