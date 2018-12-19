---
layout: post
title: "Control de brillo mediante script"
date: 2016-05-22 16:25:06 -0700
comments: true
---

En esta entrada contare como controlar el brillo desde teclado mediante un script.

Para comenzar tendremos que localizar el directorio donde están los ficheros de configuración del brillo, en mi caso esta en:
<div class="shell"><code>/sys/class/backlight/<strong>intel_backlight</strong>/</code></div>
&nbsp;

Este ultimo directorio puede variar según nuestra maquina.

Dentro encontraremos:
<pre>actual_brightness  bl_power  <strong>brightness</strong>  device  <strong>max_brightness</strong>  power	subsystem  type  uevent</pre>
Nos quedamos con max_brightness y brightness. En brightness encontramos el brillo a modificar y en max_brightness encontraremos el brillo máximo, es importante tener en cuenta este ultimo para el modificador ya que nos podemos encontrar desde 99 hasta 4882.

Para comenzar descargamos el script: <a href="https://github.com/Tedezed/Codigo-variado/tree/master/control_brillo">https://github.com/Tedezed/Codigo-variado/tree/master/control_brillo</a>

Modificamos el valor de la variable <strong>modificador</strong> a nuestro antojo y la ruta de los ficheros brightness y max_brightness.

subir_brillo.sh
```
#!/bin/sh

modificador=30

brillo=$(cat /sys/class/backlight/intel_backlight/brightness)
max_brillo=$(cat /sys/class/backlight/intel_backlight/max_brightness)
brillo=$(expr $brillo + $modificador)

if [ "$brillo" -lt "$max_brillo" ]; then
echo $brillo &gt; /sys/class/backlight/intel_backlight/brightness
else
echo $max_brillo &gt; /sys/class/backlight/intel_backlight/brightness
fi
```

bajar_brillo.sh
```
#!/bin/sh

modificador=30

brillo=$(cat /sys/class/backlight/intel_backlight/brightness)
max_brillo=$(cat /sys/class/backlight/intel_backlight/max_brightness)
brillo=$(expr $brillo - $modificador)

if [ 0 -lt "$brillo" ]; then
echo $brillo &gt; /sys/class/backlight/intel_backlight/brightness
else
echo 0 &gt; /sys/class/backlight/intel_backlight/brightness
fi
```

Para instalarlo ejecutamos el instalador:
<div class="shell"><code>sh install_root.sh</code></div>
&nbsp;

install_root.sh
```
#!/bin/sh
mkdir /usr/local/bin/control_brillo
cp subir_brillo.sh /usr/local/bin/control_brillo
cp bajar_brillo.sh /usr/local/bin/control_brillo
chmod 755 -R /usr/local/bin/control_brillo
```

Por ultimo añadimos un atajo de teclado, en Configuración -&gt; teclado.
<div class="shell"><code>sudo sh /usr/local/bin/control_brillo/subir_brillo.sh</code></div>
&nbsp;
<div class="shell"><code>sudo sh /usr/local/bin/control_brillo/bajar_brillo.sh</code></div>
&nbsp;
