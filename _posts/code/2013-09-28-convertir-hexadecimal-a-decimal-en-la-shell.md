---
layout: post
title:  "Convertir hexadecimal a decimal en la shell"
date:   2013-09-28
categories: code
comments: true
share: true
---

Estas son las formas más sencillas que he encontrado, ¿alguna sugerencia mejor?

Usando `bc`:

{% highlight shell %}
echo "ibase=16; 007B" | bc
123
{% endhighlight %}

Más sencillo aún, usando printf:

{% highlight shell %}
printf "%d\n" 0x7b
123
{% endhighlight %}

Claro, que este método no es tan general como usando `bc`
