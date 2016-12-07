---
layout: post
title:  "Cadenas en bash"
excerpt: Manejo básico de cadenas en bash
date:   2012-09-23
categories: code
comments: true
share: true
---

A veces se encuentran ciertas limitaciones a la hora de escribir scripts de shell en el manejo de cadenas y es que, muchas veces, nos olvidamos de que bash tiene un montón de operaciones sobre strings. Por ejemplo:

{% highlight shell %}
# Cadena de ejemplo:
string=123.abc.456.ABC.123

# Ver la longitud de la cadena
$ echo ${#string}
19

# Extraer subcadena desde una posicion
$ echo ${string:4}
abc.456.ABC.123

# Extraer subcadena de longitud determinada
# desde una posicion
$ echo ${string:4:7}
abc.456

# Eliminar la cadena coincidente del principio de la cadena
$ echo ${string#123}
.abc.456.ABC.123

# Eliminar la cadena coincidente del final de la cadena
$ echo ${string%123}
123.abc.456.ABC.

# Reemplazar la primera aparación de una subcadena en la cadena
$ echo ${string/123/QWE}
QWE.abc.456.ABC.123

# Reemplazar todas las apariciones de una subcadena en la cadena
$ echo ${string//123/QWE}
QWE.abc.456.ABC.QWE

# Reemplazar si hay coincidencia al pricipio o al final de la cadena:
$ echo ${string/#123/QWE}
QWE.abc.456.ABC.123

$ echo ${string/%123/QWE}
123.abc.456.ABC.QWE

# Longitud de la coincidencia al principio de la cadena (dos formas)
$ expr match "$string" '123'
3
$ expr "$string" : '123'
3
{% endhighlight %}

## Comparación de cadenas

Otra cuestión donde no es difícil cometer errores es en las comparaciones; al no ser bash un lenguaje tipado, ciertas construcciones pueden dar lugar a errores que pasan desapercibidos y llegan a ser difíciles de detectar. Así que recordemos:

**El operador de comparación es =, no -eq.**

Ejemplos:

{% highlight shell %}
$ $a=foo
$ $b=bar
$ [ "$a" = "$b" ] && echo 'las dos variables son iguales' || echo 'las dos variables son distintas'
las dos variables son distintas

$ [ "$a"="$b" ] && echo 'las dos variables son iguales' || echo 'las dos variables son distintas'
las dos variables son iguales
{% endhighlight %}

== es sinónimo de =

{% highlight shell %}
$ [ "$a" == "$b" ] && echo 'las dos variables son iguales' || echo 'las dos variables son distintas'
las dos variables son distintas
{% endhighlight %}

El operador “distinto de” es !=

{% highlight shell %}
$ if [ "$a" != "$b" ]; then echo "Las dos variables son distintas"; fi
Las dos variables son distintas
{% endhighlight %}

Pero, con dobles corchetes, el funcionamiento es distinto (pattern matching):

{% highlight shell %}
$ filename=foto.jpg
$ if [[ "$filename" != *.gif ]]; then echo "El fichero no parece un gif"; fi
El fichero no parece un gif
{% endhighlight %}

Si las expresiones son compuestas, se pueden usar los operadores -a (y lógico) y -o (o lógico). Y, sólo en el caso de usar dobles corchetes, sus “casi” equivalentes && y ||

{% highlight shell %}
$ if [ "$a" = "foo" -a "$b" = "bar" ]; then echo "foobar"; fi
$ if [[ "$a" == "foo" &amp;&amp; "$b" == "bar" ]]; then echo "foobar"; fi
{% endhighlight %}
