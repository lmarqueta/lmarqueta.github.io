---
layout: post
title:  "Cómo funciona RSA"
excerpt: "Introducción al algoritmo RSA"
date:   2017-02-04
categories: blog
tags: [security, cryptography, rsa]
image:
  feature: randomChars.png
comments: true
share: true
---
Cuando se habla de criptografía de clave pública enseguida se nos vienen a la cabeza las siglas "RSA".

RSA son las iniciales de Rivest, Shamir y Adleman, investigadores del MIT que propusieron el algoritmo RSA en 1977. La idea es la siguiente: determinadas operaciones matemáticas son muy sencillas en un sentido pero extremadamente complejas en sentido contrario; es el caso de la factorización. Dado un par de números primos "grandes" es muy fácil hayar su producto pero, conocido el producto, no existe ningún algoritmo práctico para factorizarlo en un tiempo razonable.

Como veremos, esto dependerá del tamaño de los número elegidos.

## El algoritmo RSA

Veamos cómo funciona el algoritmo con un ejemplo estúpido. Y digo estúpido porque la fortaleza del algoritmo reside en la elección de números primos de cierto tamaño y lo que vamos a hacer es elegir enteos muy pequeños para facilitar los cálculos.

Sean pues p y q dos números primos:

```
p = 89
q = 97
```

y sea N el producto de ambos `N = p·q = 8633`. Vale, esto no cuadra mucho con lo anteriormente explicado acerca de que la fortaleza de RSA está basada en la dificultad de factorizar números grandes, pero supongamos que 8633 es "grande" :)

Ahora necesitamos calcular el valor de la función φ de Euler para N:

```
φ(N) = φ(p·q) = φ(p)·φ(q) = (p-q)·(q-1)
```

Como todo el mundo sabe (:D) esta función nos da el número de enteros positivos menores o iguales que N coprimos con N. En nuestro caso valdrá `φ(N) = (89-1)·(97-1) = 8448`.

El siguiente paso es elegir un primo entre 3 y φ(N). En la vida real, este número será habitualmente 65537 (a menos que dé la casualidad que dicho número sea divisor de φ(N), que podría ocurrir, en cuyo caso habrá que elegir otro). En nuestro caso podemos elegir el 23. A este número lo llamaremos exponente público (y en adelante lo notaremos como "e").

Ahora vamos a calcular el exponente privado d, que será el inverso de e en módulo φ(N). Es decir, en nuestro ejemplo, el inverso de 23 en aritmética de módulo 8448:

```
e·d = 1 (mod φ(N))
```

Esto es así porque todos recordamos perfectamente que el inverso de un número es otro tal que multiplicado por el primero nos da 1 (y recordad que estamos trabajando en módulo φ(N), que es 60 en nuestro ejemplo).

Para este cálculo tenemos dos opciones:

1. Usar el algoritmo de Euclides extendido
2. Usar la fuerza bruta

Como hemos tomado intencionadamente números pequeños, optamos por la opción 2 y no liamos más el tema:

```
23·4775 = 109825 = 1 (mod 8448)
```

Pues ya tenemos todo lo que necesitamos y nuestro par de claves pública y privada será como sigue:

* clave pública = (8633, 23)
* clave privada = (8633, 4775)

Como vemos, las claves vienen caracterizadas por el producto de los primos elegidos inicialmente y los exponentes público y privado.

## Cifrando y descifrando mensajes

Sea m el mensaje que queremos cifrar. El mensaje cifrado será:

```
c = m^e (mod N)
```

Y para descifrar:

```
m = c^d (mod N)
```

En nuestro ejemplo, sea el mensaje `m = 1234`

Ciframos con nuestra clave pública; es más, _cualquiera_ podrá cifrar el mensaje con nuestra clave pública:

```
c = 1234^23 (mod 8633) = 5644
```

Pero solo nosotros podremos descifrarlo con nuestra clave privada:

```
5644^4775 (mod 8633) = 1234
```

## ¡¡No hagan esto en sus casas!!

Obviamente este ejemplo es estúpido porque para factorizar 8633 casi no hace falta ni la calculadora. Actualmente las claves RSA suelen tener entre 1024 y 2048 bits (siendo la longitud de la clave la de φ(N)).

Factorizar N permite obtener p y q y, por tanto, los exponentes d y e. De ahí que la dificultad en la factorización sea la clave de la seguridad de RSA. Se dice que con menos de 256 bits se puede factorizar un número en unos minutos con un PC casero. En 1999 se factorizó un número de 512 bits en un sistema distribuido trabajando varios menses, y [Benjamin Moody] lo hizo en 73 días con tan solo un procesador AMD.

Hay quien dice que las claves de 1024 bits dejarán de ser seguras a medio plazo; así que la próxima vez que crees una clave RSA con `openssl` no te limites a aceptar el valor por defecto (posiblemente 2048), sino auméntalo hasta 4096 :)

## Padding

La descripción anterior no corresponde exactamente con la implementación real del algoritmo, ya que no se han tenido en cuenta técnicas como el relleno (_padding_) empleadas para dificultar los ataques.

Por otra parte, cifrar mensajes completos con RSA no es práctico. Así, el método habitual es cifrar los mensajes con algoritmos como [AES] con una clave aleatoria y enviar al interlocutor esta clave cifrada con RSA. De este modo, la clave se transmmite por un canal seguro.

[Benjamin Moody]: http://lukenotricks.blogspot.com.es/2009/08/solo-desktop-factorization-of-rsa-512.html
[AES]: https://es.wikipedia.org/wiki/Advanced_Encryption_Standard
