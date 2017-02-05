---
layout: post
title:  "Cómo funciona RSA (II)"
excerpt: "Jugando con RSA: recuperando la clave privada del doctor Bacterio"
date:   2017-02-05
categories: blog
tags: [security, cryptography, rsa]
image:
  feature: randomChars.png
comments: true
share: true
---
En al artículo de ayer vimos una breve introducción a cómo funciona RSA. Aquí vemos algún otro ejemplo algo más elaborado.

Una mañana cualquiera en las oficinas de la T.I.A...

<figure class="half">
	<img src="/images/la-tia.jpg" alt="image">
	<img src="/images/abaco.png" alt="image">
	<figcaption>El personal de la TIA y su centro de cálculo</figcaption>
</figure>

El profesor Bacterio decide enviar un mensaje cifrado al superintendente Vicente. Para ello decide utilizar RSA pero, dadas las limitaciones del centro de cálculo de la organización, decide utilizar una clave de 128 bits:

```
openssl genrsa 128 > bacterio.key
Generating RSA private key, 128 bit long modulus
.....+++++++++++++++++++++++++++
..+++++++++++++++++++++++++++
e is 65537 (0x10001)
```

La clave se guarda en el fichero `bacterio.key` con el siguiente contenido:

```
-----BEGIN RSA PRIVATE KEY-----
MGECAQACEQDPY+Q9PydW2kM+sQX+m/VRAgMBAAECEEbw8pgIzFomvXX6DYJ6YAEC
CQDt5dDtG7d+UQIJAN8rzBLOHEcBAgg7B9mt3EDN4QIIShof0+quRwECCH60gzYC
Fxjs
-----END RSA PRIVATE KEY-----
```

El doctor Bacterio genera la correspondiente clave pública:

```
openssl rsa -in bacterio.key -pubout
writing RSA key
-----BEGIN PUBLIC KEY-----
MCwwDQYJKoZIhvcNAQEBBQADGwAwGAIRAM9j5D0/J1baQz6xBf6b9VECAwEAAQ==
-----END PUBLIC KEY-----
```

Y pone esta a disposición en un servidor público. Hecho esto, cifra el mensaje y se lo hace llegar al Super:

```
echo -n "**********"|openssl rsautl -encrypt -inkey bacterio.key|base64
chu8HWOIE2p+oztRKGiCcg==
```

He "ofuscado" el mensaje con asteriscos para que el ejercicio tenga sentido. Por desgracia, la Banda del Orzuelo interceptó el mensaje y, tras una sencilla búsqueda, obtivo la clave pública del científico.

El análisis de la clave pública mostraba lo siguiente:

```
openssl rsa -inform PEM -text -noout -pubin < bacterio.pub
Modulus (128 bit):
    00:cf:63:e4:3d:3f:27:56:da:43:3e:b1:05:fe:9b:
    f5:51
Exponent: 65537 (0x10001)
```

Viendo con natural sorpresa que la clave es de tan solo 128 bits, deciden abordar la titánica tarea de factorizar el módulo. Para ello:

```
# Convertir a decimal
echo "00:cf:63:e4:3d:3f:27:56:da:43:3e:b1:05:fe:9b:f5:51" |tr -d :
00cf63e43d3f2756da433eb105fe9bf551
python -c "python -c "print 0x00cf63e43d3f2756da433eb105fe9bf551"
275668861758325193452756783408145495377
```

Para factorizar este número se puede utilizar la librería primefac y el siguiente código:

```
$ python
>>> import primefac
>>> import sys
>>> factors = list(primefac.primefac(n))
>>> print "\n".join(map(str, factors))
16081171275595925249
17142337273446497873
```

El i5 de la banda rivar ha tardado unos 15 segundos en resolverlo. Otra opción es buscarlo en http://factordb.com. En este momento, la Banda del Orzuelo ya tiene p, q (los factores primos), N y e (el exponente público que, como bien sabían porque leyeron el artículo anterior, tenía muchas posibilidades de ser 65537). Así pues, necesitan calcular φ(N), lo cual es trivial porque tienen p y q:

```
>>> print (16081171275595925249-1)*(17142337273446497873-1)
275668861758325193419533274859103072256
```

Para obtener `d` hay que calcular el inverso modular de `e` en módulo φ(N). Utilizaremos este código:

```python
def extended_gcd(aa, bb):
    lastremainder, remainder = abs(aa), abs(bb)
    x, lastx, y, lasty = 0, 1, 1, 0
    while remainder:
        lastremainder, (quotient, remainder) = remainder, divmod(lastremainder, remainder)
        x, lastx = lastx - quotient*x, x
        y, lasty = lasty - quotient*y, y
    return lastremainder, lastx * (-1 if aa < 0 else 1), lasty * (-1 if bb < 0 else 1)


def modinv(a, m):
	g, x, y = extended_gcd(a, m)
	if g != 1:
		raise ValueError
	return x % m

print modinv(65537, 275668861758325193419533274859103072256)
```

Al ejecutarlo obtenemos:

```python
print modinv(65537, 275668861758325193419533274859103072256)
94297031339520182279919693543973216257

# En hexadecimal:
$ python -c "print hex(94297031339520182279919693543973216257)"
0x46f0f29808cc5a26bd75fa0d827a6001L

```

Tenemos (bueno, la banda del Orzuelo lo tiene) todo lo necesario para reconstruir la clave privada del profesor Bacterio. El procedimiento sigue así (con código tomado de [0day.work] y [crypto.stackexchange.com])

```
python
>>> e = 65537
>>> n = 16081171275595925249*17142337273446497873
>>> p = 16081171275595925249
>>> q = 17142337273446497873
>>> phi = (p -1)*(q-1)

>>> def extended_gcd(aa, bb):
...     lastremainder, remainder = abs(aa), abs(bb)
...     x, lastx, y, lasty = 0, 1, 1, 0
...     while remainder:
...         lastremainder, (quotient, remainder) = remainder, divmod(lastremainder, remainder)
...         x, lastx = lastx - quotient*x, x
...         y, lasty = lasty - quotient*y, y
...     return lastremainder, lastx * (-1 if aa < 0 else 1), lasty * (-1 if bb < 0 else 1)
...
>>>
>>> def modinv(a, m):
...     g, x, y = extended_gcd(a, m)
...     if g != 1:
...             raise ValueError
...     return x % m
...
>>> dp = modinv(e,(p-1))
>>> dq = modinv(e,(q-1))
>>> qi = modinv(q,p)
>>> d = modinv(e,phi)
>>>
>>> import pyasn1.codec.der.encoder
>>> import pyasn1.type.univ
>>> import base64
>>>
>>> def pempriv(n, e, d, p, q, dP, dQ, qInv):
...     template = '-----BEGIN RSA PRIVATE KEY-----\n{}-----END RSA PRIVATE KEY-----\n'
...     seq = pyasn1.type.univ.Sequence()
...     for x in [0, n, e, d, p, q, dP, dQ, qInv]:
...         seq.setComponentByPosition(len(seq), pyasn1.type.univ.Integer(x))
...     der = pyasn1.codec.der.encoder.encode(seq)
...     return template.format(base64.encodestring(der).decode('ascii'))
...
>>> key = pempriv(n,e,d,p,q,dp,dq,qi)
>>> key
'-----BEGIN RSA PRIVATE KEY-----\nMGECAQACEQDPY+Q9PydW2kM+sQX+m/VRAgMBAAECEEbw8pgIzFomvXX6DYJ6YAECCQDfK8wSzhxH\nAQIJAO3l0O0bt35RAghKGh/T6q5HAQIIOwfZrdxAzeECCGhPNZy2JiqG\n-----END RSA PRIVATE KEY-----\n'
>>> f = open("recovered.key","w")
>>> f.write(key)
>>> f.close
```

Con la clave recuperada (`recovered.key`) y el mensaje secreto interceptado hacemos lo siguiente y...

```
$ echo "chu8HWOIE2p+oztRKGiCcg=="|base64 -D|openssl rsautl -decrypt -inkey recovered.key
hola
```

Et voilà! El mensaje secreto era: **"hola"**.

## Limitación del tamaño del mensaje

Puede que el mensaje haya sido un poco decepcionante, pero si se intente cifrar con RSA un mensaje largo ocurrirá esto:

```
echo -n "mensaje secretísimo" |openssl rsautl -encrypt -inkey bacterio.key
RSA operation error
1473:error:0406D06E:rsa routines:RSA_padding_add_PKCS1_type_2:data too large for key size:/BuildRoot/Library/Caches/com.apple.xbs/Sources/OpenSSL098/OpenSSL098-64.30.2/src/crypto/rsa/rsa_pk1.c:153:
```

RSA no está diseñado para cifrar mensajes arbitrariamente largos; el límite depende de la longitud del módulo menos el padding, lo cual no deja realmente muchos bits. Habitualmente el mensaje se cifra con un algoritmo de cifrado simétrico (AES por ejemplo) y con RSA se cifra la clave.

[0day.work]: https://0day.work/how-i-recovered-your-private-key-or-why-small-keys-are-bad/
[crypto.stackexchange.com]: http://crypto.stackexchange.com/questions/25498/how-to-create-a-pem-file-for-storing-an-rsa-key
