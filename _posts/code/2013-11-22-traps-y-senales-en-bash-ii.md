---
layout: post
title:  Traps y señales en bash (II)
excerpt: segunda parte del artículo sobre señales en bash
date:   2013-11-22
categories: code
comments: true
share: true
tags: [bash, shell, traps]
---

Continuando con el artículo anterior, vamos a ver un ejemplo de cómo impedir la ejecución de varias instancias de un script.

Sea este complejísimo script, que cuenta de 1 en 1 hasta 10:

{% highlight shell %}
#!/bin/bash
i=1
while [ $i -le 10 ]; do
 echo $i
 i=$((i+1))
 sleep 1
done
{% endhighlight %}

Evidentemente, no hay problema en ejecutarlo tantas veces como sea posible de forma concurrente. Por ejemplo:

{% highlight shell %}
$ ./nobloqueo.sh & ./nobloqueo.sh
[1] 19585
1
1
2
2
3
3
^C
{% endhighlight %}

Uf, qué lío. Así no hay quien aprenda a contar. Sería mucho mejor impedir que el script se ejecutase más de una vez. Para ello, el método “clásico” es escribir un fichero de “lock”. Si el fichero existe, no me ejecuto; si no existe, lo escribo, me ejecuto, y lo borro después, para dejar que otro usuario pueda ejecutarlo. Hágase:

{% highlight shell %}
$ cat bloqueo.sh
#!/bin/bash
LOCK="$HOME/bloqueo.lck"
# Si no existe el fichero, lo escribo y me ejecuto:
if [ ! -e $LOCK ]; then
 touch $LOCK
 i=1
 while [ $i -le 10 ]; do
 echo $i
 i=$((i+1))
 sleep 1
 done
 /bin/rm $LOCK
else
 echo "Ya estoy contando hasta 10"
fi
{% endhighlight %}

Si el fichero bloqueo.lck no existe, se creará y comenzará la cuenta; pero si existe dicho fichero, el script se quejará y no contará nada. Para probarlo, ejecuto el script en un terminal…

{% highlight shell %}
$ ./bloqueo.sh
1
2
3
{% endhighlight %}

y, mientras, corre, me voy a otro terminal a tratar de ejecutarlo de nuevo:

{% highlight shell %}
$ ./bloqueo.sh
Ya estoy contando hasta 10
{% endhighlight %}

¿Correcto? ¡¡Pues no, muy mal!! Aquí se da lo que se conoce como *race condition*; y es que en el intervalo entre que se chequea la existencia del fichero y se crea este, es posible que otro script empiece a ejecutarse y se venga abajo todo nuestro tinglado. Claro, la operación no es atómica. ¿No crees que sea posible? Mira qué fácil:

{% highlight shell %}
$ ./bloqueo.sh & ./bloqueo.sh
[1] 20099
1
1
2
2
3
3
4
4
{% endhighlight %}

Bash ofrece un mecanismo que sirve de ayuda para estos casos: la opción noclobber. Veamos qué dice man bash:

> If the redirection operator is >, and the noclobber option to the set builtin has been enabled, the redirection will fail if the file whose name results from the expansion of word exists and is a regular file. If the redirection operator is >|, or the redirection operator is > and the noclobber option to the set builtin command is not enabled, the redirection is attempted even if the file named by word exists.

Vamos, que si está establecido noclobber y se usa el operador de redirección >, esta fallará si el fichero existe. Probemos así entonces:

{% highlight shell %}
$ cat bloqueo2.sh
#!/bin/bash
LOCK="$HOME/bloqueo.lck"
# Si no existe el fichero, lo escribo y me ejecuto:
if ( set -o noclobber; echo "$$" > "$LOCK") 2> /dev/null; then
 i=1
 while [ $i -le 10 ]; do
 echo $i
 i=$((i+1))
 sleep 1
 done
 /bin/rm $LOCK
else
 echo "Ya estoy contando hasta 10"
fi
{% endhighlight %}

Y probando a ejecutarlo como antes:

{% highlight shell %}
$ ./bloqueo2.sh & ./bloqueo2.sh
[1] 20178
1
Ya estoy contando hasta 10
$ 2
3
4
5
{% endhighlight %}

Vemos como, efectivamente, uno de los dos scripts falla.

Si nos cansamos de ver cómo cuenta el script y lo interrumpimos (Ctrl+C), ¿qué es lo que ocurrirá? Que la siguiente vez que queramos lanzarlo no podremos, porque nos habremos dejado colgando el fichero de lock. Así que, poniendo en práctica lo visto en el artículo referenciado arriba, podemos hacer esto:

{% highlight shell %}
$ cat bloqueo3.sh
#!/bin/bash
LOCK="$HOME/bloqueo.lck"
trap 'rm -f "$LOCK"; exit' INT TERM EXIT ERR
# Si no existe el fichero, lo escribo y me ejecuto:
if ( set -o noclobber; echo "$$" & "$LOCK") 2> /dev/null; then
 i=1
 while [ $i -le 10 ]; do
 echo $i
 i=$((i+1))
 sleep 1
 done
 /bin/rm $LOCK
else
 echo "Ya estoy contando hasta 10"
fi
{% endhighlight %}

¿Mejor, no? ¡¡Pues no!! ¡¡Muchísimo peor!! Definiendo así el trap, un segundo script que se ejecutara no contaría, pero borraría el fichero de lock, permitiendo que se ejecutase un tercer script. La mejor alternativa sería esta:

{% highlight shell %}
$ cat ./bloqueo2.sh
#!/bin/bash
LOCK="$HOME/bloqueo.lck"
# Si no existe el fichero, lo escribo y me ejecuto:
if ( set -o noclobber; echo "$$" & "$LOCK") 2> /dev/null; then
 trap 'rm -f "$LOCK"; exit' INT TERM EXIT ERR
 i=1
 while [ $i -le 10 ]; do
 echo $i
 i=$((i+1))
 sleep 1
 done
 /bin/rm $LOCK
 trap - INT TERM EXIT ERR
else
 echo "Ya estoy contando hasta 10"
fi
{% endhighlight %}

De esta forma, en el momento en que el script falla aún no se han redefinido los traps y el fichero .lck no se borra.

Aun así, no estaría de más comprobar antes de borrar el fichero de lock que realmente ha sido escrito por mí (de ahí que se escriba el pid en el fichero). Pero bueno, eso ya es segundo de bash
