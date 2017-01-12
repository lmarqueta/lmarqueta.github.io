---
layout: post
title:  "Ubuntu en el ThinkPad"
excerpt: "Instalación de Ubuntu 16.04 LTS en un ThinkPad Edge 330"
date:   2017-01-12
categories: blog
tags: [ubuntu, linux, thinkpad]
image:
  feature: ubuntu.jpg
comments: true
share: true
---
Por aquéllo de empezar el año con alguna novedad, me da por cambiar mi vieja Debian en el ThinkPad Edge 330. Después de tantos años con Debian estable, el desktop puede resultar un poco aburrido.

La verdad es que la primera idea fue actualizar a _unstable_ para ver las novedades, ya que no queda tanto para la congelación y publicación de lo que será la versión 9 _Stretch_ pero _hubo un percance_ durante la actualización y tuve que empezar de cero.

Ya puestos, y por razones que no vienen al caso, lo hice con Ubuntu... y con Unity. Obviamente, acabé tratando de configurar Unity de forma que se comportase parecido a Gnome, al menos ciertos gestos que ya tengo muy interiorizados. Y la verdad es que no cuesta demasiado trabajo.

## Post-instalación

### Paquetes adicionales

La instalación "normal" es muy completa y no requiera la instalación de demasiados paquetes; unas pocas aplicaciones que uso habitualmente y recuperar de las copias de seguridad los ajustes correspondientes.

```
# General
sudo apt install vim keepass2 hamster-applet build-essential unison-gtk keepnote irssi tmux mutt network-manager-openvpn-gnome cryptkeeper enigmail
# Mapear unidades de red CIFS
sudo apt install smbclient winbind python-crypto-dbg python-crypto-doc heimdal-clients
```

Para que la notificación de `hamster` en el panel superior muestre la actividad en curso hay que activar una opción:

```
gconftool-2 --set "/apps/hamster-indicator/show_label" --type bool "true"
```

### Java

Uso el de Oracle porque el OpenJDK me ha dado algún problema con alguna aplicación muy particular:
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

### Mensajería

Sky: ver https://tel.red/repos.htm#debian_cli

```
sudo sh -c 'echo deb https://tel.red/repos/ubuntu xenial non-free >/etc/apt/sources.list.d/telred.list'
cat/etc/apt/sources.list.d/telred.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 9454C19A66B920C83DDF696E07C8CCAFCE49F8C5
sudo apt update
sudo apt install sky
```

Skype: descargar la versión de Ubuntu de https://www.skype.com/en/download-skype/skype-for-linux/downloading/?type=ubuntu64, instalar con `dpkg` e instalar dependencias con `apt -f install`

## Configuración de Unity

### Hot corners

Curiosamente, me encuentro tratando de replicar el comportamiento de Gnome Shell; tengo demasiado interiorizado el _"ratón a la esquina superior izquierda"_. Y, en principio, no hay opción de lanzar el `dash` de esta forma, pero tiene solución. Hay que instalar el paquetito `xdotool` que permite generar o "simular" eventos de teclado y ratón. En este caso necesitamos simular la pulsación de la tecla "Super" o "Windows" (aunque en mi caso tiene esta pinta:)

![Tecla Super]({{ site.url }}/images/super_l.jpg)

Hará falta un paquete más para acceder a la configuración "fina" de compiz: `compizconfig-settings-manager`. Nos aseguramos en los _settings_ de que la esquina superior izquierda no tiene ninguna actividad asignada, creamos en _"Commands"_ el comando que enviaremos con `xdotool` y lo asignamos en la pestaña _"Edge bindings"_ a la esquina superior izquierda. Paso 1:

![Paso 1]({{ site.url }}/images/compiz-1.png)

Paso 2:

![Paso 2]({{ site.url }}/images/compiz-2.png)

### Arranque de aplicaciones

Añadir unas pocas aplicaciones al arranque de sesión:

![Startup Applications]({{ site.url }}/images/startupapp.png)

### Jekyll

¿Cómo escribir esto si no? :)

```
sudo apt-get install ruby ruby-dev make gcc
sudo gem install jekyll bundler
```
