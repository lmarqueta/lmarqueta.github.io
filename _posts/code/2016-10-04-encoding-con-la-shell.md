---
layout: post
title: "Encoding con la Shell"
excerpt: "Shell _snippets_ para distintas codificaciones"
date: 2016-10-04
modified: 2016-12-09
categories: code
image:
  feature: bluebinary.jpg
comments: true
share: true
---
_Oneliners_ para distintos _encodings_ con la shell

* Table of contents
{:toc}

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

## rev

(Coño, no me sabía esta O_o)

```shell
echo "hola"|rev
aloh
```

## Hexdump
```shell
echo "lorem ipsum dolor sit amet" | xxd -p
6c6f72656d20697073756d20646f6c6f722073697420616d65740a
```

Y al revés...

```shell
echo 6c6f72656d20697073756d20646f6c6f722073697420616d65740a | xxd -p -r
lorem ipsum dolor sit amet
```

Pero con `hexdump` hay muchas más opciones (convertir a octal, a binario...):

```shell
echo "lorem ipsum dolor sit amet" | hexdump -v -e '/1 "%03i "'; echo
108 111 114 101 109 032 105 112 115 117 109 032 100 111 108 111 114 032 115 105 116 032 097 109 101 116 010

echo "lorem ipsum dolor sit amet" | hexdump -v -e '/1 "%02x "'; echo
6c 6f 72 65 6d 20 69 70 73 75 6d 20 64 6f 6c 6f 72 20 73 69 74 20 61 6d 65 74 0a

echo "lorem ipsum dolor sit amet" | hexdump -v -e '/1 "%03o "'; echo
154 157 162 145 155 040 151 160 163 165 155 040 144 157 154 157 162 040 163 151 164 040 141 155 145 164 012
```

## openssl
```shell
echo "s3cr3t" | openssl passwd -crypt -stdin -salt foobar # salt foobar
echo "s3cr3t" | openssl passwd -crypt -stdin              # random salt
echo "s3cr3t" | openssl passwd -1 -stdin -salt foobar     # shadow
echo -n "s3cr3t" | openssl passwd -apr1 -stdin -salt foobar # apache
```

Y muchos más [aquí](https://gist.github.com/Janfy/940195a6fc4278112458]) y [aquí](https://www.redspin.com/it-security-blog/2009/07/string-encoding-in-the-shell/).
