---
layout: post
title:  "Encoding con la shell"
date:   2016-10-04
categories: bash
---

# Encoding con la Shell

## Rot13

```
echo "blah blah blah" | tr '[a-m][n-z][A-M][N-Z]' '[n-z][a-m][N-Z][A-M]'
oynu oynu oynu
```

Por sus propias características, aplicando el mismo algoritmo "desofusca" la cadena:

```
echo "oynu oynu oynu" | tr '[a-m][n-z][A-M][N-Z]' '[n-z][a-m][N-Z][A-M]'
blah blah blah
```

## Base64

```
echo echo "blah blah blah" | base64 
YmxhaCBibGFoIGJsYWgK
```

Y al revés:

```
echo YmxhaCBibGFoIGJsYWgK | base64 -d 
blah blah blah
```

## Hexdump
