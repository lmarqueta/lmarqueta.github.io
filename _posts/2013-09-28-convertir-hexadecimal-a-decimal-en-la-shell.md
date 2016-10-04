---
layout: post
title:  "Convertir hexadecimal a decimal en la shell"
date:   2013-09-28
categories: bash
---

Estas son las formas más sencillas que he encontrado, ¿alguna sugerencia mejor?

Usando `bc`:

```
echo "ibase=16; 007B" | bc
123
```

Más sencillo aún, usando printf:

```
printf "%d\n" 0x7b
123
```

Claro, que este método no es tan general como usando `bc`
