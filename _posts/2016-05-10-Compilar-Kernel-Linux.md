---
layout: post
title: "Compilar Kernel Linux"
date: 2016-05-10 16:25:06 -0700
comments: true
---

En esta entrada tratare el tema de la compilación del kernel linux en un OS Debian Jessie, donde intentare explicarlo de una forma fácil, simple y rápida. Espero que os sirva como una introducción.

En primer lugar necesitaremos una serie de paquetes para poder compilar nuestro kernel:
```apt-get install build-essential make ncurses-dev libqt4-dev pkg-config```

Podremos ver la información del kernel con:
```uname -r```

Descargamos el codigo fuente del kernel Linux:
```apt-get install linux-source```

El resultado es el fichero /usr/src/linux-source-3.16.tar.xz

Creamos la siguiente carpeta carpeta:
```mkdir ~/linux```
```cd ~/linux```

Descomprimimos el fichero anterior:
```tar xJf /usr/src/linux-source-3.16.tar.xz```

Utilizar el fichero de configuración del núcleo actual como punto de partida. Para ello:
```cp /boot/config-`uname -r` ~/linux/linux-source-3.16/.config```
```cd linux-source-3.16```

Dejamos el kernel con los módulos esenciales de nuestra maquina:
```make localmodconfig```

Especificamos la versión de la compilación que queramos realizar:
```nano Makefile```

Con cada compilacion cambiamos la version:
```
EXTRAVERSION =-n0001
---
EXTRAVERSION =-n0002
```

También podremos configurarlo y personalizarlo con:
```make xconfig```

Compilar con: (`[-j numero_de_hilos]`, para forzar los cores)
```make -j 8 deb-pkg```

Instalamos el kernel compilado con:
```dpkg -i linux-image-3.16.7-new0001_3.16.7-new0001-1_amd64.deb```

Actualizamos grup para añadir nuestro kernel personalizado:
```update-grub```

Para limpiar y recompilar:
Simple:
```make clean```

Profundo:
```make mrproper```

Eliminar kernel instalado:
```dpkg -r linux-image-3.16.7-new0001```

Intentare añadir nuevas instrucciones interesantes en la entrada a lo largo del tiempo.

Un saludo!