---
layout: post
title:  Cómo ampliar discos cifrados en macOS Sierra
excerpt: Cómo ampliar discos cifrados en macOS Sierra
date:   2017-07-07
categories: blog
image:
  feature: encrypt.jpeg
comments: true
share: true
tags: [macos]
---
Aun teniendo el disco completo cifrado con [File Vault 2], en ocasiones es recomendable utilizar un disco "virtual" cifrado dentro del mismo disco. Aunque hay varias alternativas, desde hace tiempo (posiblemente desde Mountain Lion) es posible hacer esto con herramientas nativas de macOS sin necesidad de recurrir a aplicaciones de terceros.

Efectivamente, es posible hacer esto directamente con el [Disk Utility]. El proceso es muy sencillo:

* Crear una carpeta -o utilizar una existente-
* Arrancar el Disk Utility
* En la barra de menú, seleccionar:
	* File -> New image -> Image from folder...
* Seleccionar la carpeta que queremos cifrar
* Indicar el tipo de cifrado (AES 128 o 256). En general, 128 será suficiente... o no, depende del contenido. Proporcionar la contraseña o passphrase
* Indicar el tipo de imagen (read/write u otro, en función del caso de uso).

A partir de ahí se puede montar como si se tratase de una unidad más; eso sí, al montarla, el sistema solicitará la passphrase.

![Seleccionar el tipo de volumen]({{ site.url }}/images/diskUtility1.png)

![Seleccionar el tipo de cifrado]({{ site.url }}/images/diskUtility2.png)

<h2>Mierda, se ha llenado el disco</h2>

Las unidades creadas de esta forma parecen no crecer dinámicamente; es decir, puede darse el caso de que enseguida llenemos la unidad. En principio, la misma utilidad de discos proporciona herramientas para aumentar el tamaño del volumen pero, al menos en mi caso, fallaba con un error.

Sin embargo, es muy sencillo realizar la operación desde la línea de comandos. Por ejemplo, vamos a aumentar en 250 MB el tamaño del volumen `Encrypted`:

```
hdiutil resize -size 250m /Users/me/Encrypted/Encrypted.dmg
```

[File Vault 2]: https://support.apple.com/es-es/HT204837
[Disk Utility]: https://www.lifewire.com/using-os-xs-disk-utility-2260088
