---
layout: post
title:  Gnome keyring
excerpt: Algunos usos para el keyring de Gnome
date:   2016-12-12
categories: code
comments: true
share: true
tags: [gnome, python]
---
Entre los cambios de password rutinarios hay uno que me molesta sobremanera porque no podía _escriptarlo_: el del [Gnome keyring]

El caso es que, al no poder realizar el cambio desde la línea de comandos, no tenía más remedio que arrancar la consola gráfica ([Seahorse]) y cambiarlo en ella, que es mucho más incómodo.

Finalmente he econtrado la información necesaria para resolverlo. Existe una librería en Python (gnomekeyring) no muy bien documentada; pero en Python, una vez cargado un módulo, se pueden consultar las funciones disponibles:

```shell
$ python
Python 2.7.9 (default, Jun 29 2016, 13:08:31) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import gnomekeyring
>>> help('gnomekeyring')
```

En la sección *FUNCTIONS* de la ayuda lo primero que encontramos es:

```shell
FUNCTIONS
    change_password_sync(...)
    
    create_sync(...)
    
    daemon_set_display_sync(...)
```

Como no tengo ni idea de cómo funciona, pues voy probando...

```python
>>> gk.change_password_sync()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Required argument 'keyring' (pos 1) not found
>>> gk.change_password_sync('')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Required argument 'original' (pos 2) not found
>>> gk.change_password_sync('','')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Required argument 'password' (pos 3) not found
>>> gk.change_password_sync('','','')
process 15815: arguments to dbus_message_iter_append_basic() were incorrect, assertion "_dbus_check_is_valid_path (*string_p)" failed in file ../../dbus/dbus-message.c line 2681.
This is normally a bug in some application using the D-Bus library.
Gkr-Message: secret service operation failed: Method "ChangeWithMasterPassword" with signature "(oayays)(oayays)" on interface "org.gnome.keyring.InternalUnsupportedGuiltRiddenInterface" doesn't exist

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
gnomekeyring.IOError
```

Fenomenal, hemos encontrado que la función change_password_sync parece hacer lo que yo quiero y que tiene tres argumentos:

1. keyring
2. original
3. password

¡Lo tenemos! Ya solo queda construir un pequeño script que cambie la contraseña del keyring sin neecesidad de arrancar el Seahorse. Será algo así:

```python
#!/usr/bin/python

from getpass import getpass
import gnomekeyring as gk

def is_valid(password):
    # Validate password complexity requirements
    [...]
    return True

if __name__ == '__main__':
    new = None
    cur = getpass("Please enter current Gnome keyring password: ")

    while not is_valid(new):
        new = getpass("New password: ")

    try:
        gk.change_password_sync('login', cur, new)
        print ("Password successfully changed")
    except Exception as e:
        print ("Error changing Gnome Keyring Password")
        print str(e)
```

[Gnome keyring]: https://wiki.gnome.org/Projects/GnomeKeyring
[Seahorse]: https://wiki.gnome.org/Apps/Seahorse
