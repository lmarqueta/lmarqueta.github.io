---
layout: post
title: "Encoding con la Shell"
excerpt: "Shell _snippets_ para distintas codificaciones"
categories: code
image:
  feature: bluebinary.jpg
comments: true
share: true
---
## Rot13

{% highlight shell %}
echo "blah blah blah" | tr '[a-m][n-z][A-M][N-Z]' '[n-z][a-m][N-Z][A-M]'
oynu oynu oynu
{% endhighlight %}

Por sus propias características, aplicando el mismo algoritmo "desofusca" la cadena:

{% highlight shell %}
echo "oynu oynu oynu" | tr '[a-m][n-z][A-M][N-Z]' '[n-z][a-m][N-Z][A-M]'
blah blah blah
{% endhighlight %}

## Base64

{% highlight shell %}
echo echo "blah blah blah" | base64 
YmxhaCBibGFoIGJsYWgK
{% endhighlight %}

Y al revés:

{% highlight shell %}
echo YmxhaCBibGFoIGJsYWgK | base64 -d 
blah blah blah
{% endhighlight %}

## Hexdump
