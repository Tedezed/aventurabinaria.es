---
layout: post
title: "Compilar Kernel Linux"
date: 2016-05-05 16:25:06 -0700
comments: true
---

En esta entrada tratare el tema de la compilación del kernel linux en un OS Debian Jessie, donde intentare explicarlo de una forma fácil, simple y rápida. Espero que os sirva como una introducción.

En primer lugar necesitaremos una serie de paquetes para poder compilar nuestro kernel:
<pre>apt-get install build-essential make ncurses-dev libqt4-dev pkg-config</pre>
Podremos ver la información del kernel con:
<pre>uname -r</pre>
Descargamos el codigo fuente del kernel Linux:
<pre>apt-get install linux-source</pre>
El resultado es el fichero /usr/src/linux-source-3.16.tar.xz

Creamos la siguiente carpeta carpeta:
<pre>mkdir ~/linux</pre>
<pre>cd ~/linux</pre>
Descomprimimos el fichero anterior:
<pre>tar xJf /usr/src/linux-source-3.16.tar.xz</pre>
Utilizar el fichero de configuración del núcleo actual como punto de partida. Para ello:
<pre>cp /boot/config-`uname -r` ~/linux/linux-source-3.16/.config</pre>
<pre>cd linux-source-3.16</pre>
Dejamos el kernel con los módulos esenciales de nuestra maquina:
<pre>make localmodconfig</pre>
Especificamos la versión de la compilación que queramos realizar:
<pre>nano Makefile</pre>
Con cada compilacion cambiamos la version:
<pre>EXTRAVERSION =-n0001
---
EXTRAVERSION =-n0002</pre>
También podremos configurarlo y personalizarlo con:
<pre>make xconfig</pre>
Compilar con: (<code>[-j numero_de_hilos]</code>, para forzar los cores)
<pre>make -j 8 deb-pkg</pre>
Instalamos el kernel compilado con:
<pre>dpkg -i linux-image-3.16.7-new0001_3.16.7-new0001-1_amd64.deb</pre>
Actualizamos grup para añadir nuestro kernel personalizado:
<pre>update-grub</pre>
Para limpiar y recompilar:
Simple:
<pre>make clean</pre>
Profundo:
<pre>make mrproper</pre>
Eliminar kernel instalado:
<pre>dpkg -r linux-image-3.16.7-new0001</pre>
Intentare añadir nuevas instrucciones interesantes en la entrada a lo largo del tiempo. Un saludo!