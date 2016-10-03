---
layout: post
title:  "Traps y señales en bash (I)"
date:   2013-11-21
categories: bash
---

# Traps y señales en bash (I)

Algunos  problemas típicos a la hora de crear scripts en bash son estos:

* Crear ficheros de lock para, por ejemplo, evitar la ejecución de más de una instancia del script
* Hacer limpieza de ficheros temporales si el script acaba inesperadamente

Estos y muchos más que no se me ocurren ahora se pueden resolver gracias al *built-in* `trap`. Este comando permite capturar ciertas señales y definir la acción que se realiza cuando son recibidas por el programa.

Obviamente, hay ciertas señales que no se pueden redefinir como son:

```
 9) SIGKILL
18) SIGCONT
19) SIGSTOP
```

La lista se puede ver con el comando:

{% highlight shell %}
$ kill -l
 1) SIGHUP   2) SIGINT   3) SIGQUIT  4) SIGILL   5) SIGTRAP
 6) SIGABRT  7) SIGBUS   8) SIGFPE   9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG  24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF 28) SIGWINCH    29) SIGIO   30) SIGPWR
31) SIGSYS  34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
{% endhighlight %}

Algunas señales particularmente interesantes para los casos arriba indicados son SIGHUP, SIGINT, SIGQUIT y SIGTERM:

Señal|Descripción
-----|-----------
SIGHUP|Históricamente era la señal que indicaba que el terminal al otro lado de la línea serie había “colgado”. Actualmente indica que el terminal controller se ha cerrado y suele redefinirse para recargar configuración y reabrir ficheros de log
SIGINT|Interrupción, típicamente Ctrl+C en el teclado
SIGQUIT|Enviado por el controlling terminal para terminar un programa (¿y generar un core dump?
SIGTERM|Similar a SIGINT, indica la terminación de un programa

Pues bien, estas señales se pueden capturar, redefinir o ignorar. Por ejemplo, podemos redefinir la acción que se ejecuta por defecto al recibir la señal que se genera al pulsar Ctrl+C  y decidir que, en lugar de terminar el programa, nos muestre un simpático mensaje:

{% highlight shell %}
$ cat ctrlc.sh
#!/bin/bash
trap 'echo "Paso de ti..."' SIGINT
while true; do
 sleep 1
done
{% endhighlight %}

Si lo ejecutamos y tratamos de interrumpir el programa con Ctrl+C...

{% highlight shell %}
$ chmod 755 ctrlc.sh
luis@e330:~$ ./ctrlc.sh
^CPaso de ti...
^CPaso de ti...
^CPaso de ti...
{% endhighlight %}

Mmmm... ahora, claro, habrá que ingeniárselas para parar el programa de otra forma. Esto quiere decir que hay que ser cuidadoso redefiniendo señales.

El formato del comando trap es el siguiente:

{% highlight shell %}
trap 'lista de comandos' SEÑAL1 [SEÑAL2 ...]
{% endhighlight %}

Además, la señal se puede ignorar...

{% highlight shell %}
trap '' SIGNAL
{% endhighlight %}

o resetear a su valor por defecto (definido en signal.h):

{% highlight shell %}
trap - SIGNAL
{% endhighlight %}

Con esto ya podemos ir haciendo cosas; si tenemos un script que genera ficheros temporales que queremos eliminar si termina de forma inesperada, podemos redefinir ciertas señales de este modo:


{% highlight shell %}
#!/bin/bash
#
# Borrar fichero temporal:
trap '/bin/rm /tmp/mitemp.$$; exit' SIGHUP SIGINT SIGQUIT SIGTERM
{% endhighlight %}

Rizando, el rizo, se podría añadir la pseudoseñal ERR. Si ERR está en la lista de señales, la lista de comandos indicada definida en el trap se ejecutará si se produce un error (código de salida no-cero) en el script, salvo en estas circunstancias:

* El comando que ha fallado forma parte de una lista que sigue a `while` o `until`
* Es parte de un test en una sentencia `if`
* Es parte de una lista de comandos `&&` o `||` 
* La salida del comando está invertida via `!`

Así, por ejemplo:

{% highlight shell %}
$ cat err.sh
#!/bin/bash -e
# Defino el trap
trap 'echo "Se ha producido un error"' ERR
touch testfile.txt
chmod 444 testfile.txt
echo "esto va a fallar" >testfile.txt
echo "esto no se va a ejecutar"
{% endhighlight %}

Cuando se ejecuta ocurre esto:

{% highlight shell %}
$ ./err.sh
./err.sh: line 7: testfile.txt: Permission denied
Se ha producido un error
{% endhighlight %}

Nótese que, al ejecutarse bash con la opción -e, termina tras el error y no llega a ejecutarse la última línea. Esto me recuerda que algún día habrá que escribir sobre los modificadores de bash, esos grandes desconocidos :)

Hay otra pseudoseñal interesante, EXIT, que se dispara cuando el script termina.

Y pensaba terminar con lo de los ficheros de lock, pero se quedará para una segunda parte.
