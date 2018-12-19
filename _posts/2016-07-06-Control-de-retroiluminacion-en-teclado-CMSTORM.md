---
layout: post
title: "Control de retroiluminación en teclado CMSTORM"
date: 2016-07-06 16:25:06 -0700
comments: true
---

Creamos el archivo <code>/home/xerrot/script/key</code> con un 0 o 1 en su interior.

0 quiere decir que la retroiluminación de nuestro teclado esta apagada.
1 quiere decir que la retroiluminación de nuestro teclado esta encendida.

Creamos el siguiente script para el control de la retroiluminación del teclado:
```
#!/bin/bash

key=`cat /home/xerrot/script/key`
if [ "$key" = "1" ]; then
  xset -led 3
  echo 0 &gt; /home/debian/script/key
else
  xset led 3
  echo 1 &gt; /home/debian/script/key
fi
```

Por ultimo añadimos un atajo de teclado para ejecutar el script cada vez que pulsemos a tecla de retroiluminación.

Para dejarlo fino y que se encienda el teclado automáticamente al iniciar gnome para poder ver las teclas a la hora de introducir la contraseña en el inicio de sesión, haremos lo siguiente.

Editamos el Init de Gnome `/etc/gdm3/Init/Default` introducimos justo debajo de 

	PATH="/usr/bin:$PATH"
	OLD_IFS=$IFS

Quedando así

	PATH="/usr/bin:$PATH"
	OLD_IFS=$IFS

	xset led on

Con esto ya tendríamos terminado la configuración de nuestro teclado.



