---
layout: post
title: "Pantalla LCD 16x2 y PCF8574 (I2C) para Raspberry pi"
date: 2016-08-2 16:25:06 -0700
comments: true
---

En esta entrada trataremos de utilizar una de las pantallas LCD más comunes, comunicándonos con un circuito PCF8574, mediante I2C. Estos dos últimos los compre juntos para simplificar el control de la pantalla LCD, ya que también podemos conectar la pantalla directamente a nuestra Raspberry pi, pero tendremos el inconveniente de que nos ocupara gran cantidad de las salidas disponibles. De este modo ocuparemos un total de 4 pines.

Comenzamos, actualizamos nuestro Raspbian con las siguientes instrucciones:
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
$ sudo rpi-update
```

Introducimos en /boot/config.txt la siguiente linea:
```
device_tree=
```

Reiniciamos nuestra Raspberry pi.
```
$ sudo reboot
```

Verificamos que podemos ejecutar:
```
$ sudo hwclock
```

Ejecutamos **sudo raspi-config** para cambiar la configuración de nuestra Raspberry pi.

Seleccionamos lo siguiente:
- 8 Advanced Options
- A7 I2C
- Yes o Ok
Salimos con Finish.

Instalamos paquetes necesarios:
```
$ sudo apt-get update
$ sudo apt-get install python-smbus i2c-tools
```

Editamos el archivo:
```
$ sudo nano /etc/modprobe.d/raspi-blacklist.conf
```

Este lo dejamos de la siguiente forma:
```
blacklist spi-bcm2708
#blacklist i2c-bcm2708
```

Editamos el achivo:
```
$ nano /etc/modules
```

Este lo dejamos de la siguiente forma:
```
snd-bcm2835
i2c-dev
```

Reiniciamos la Raspberry.
```
$ sudo reboot
```

Para testear introducimos el comando i2cdetectcon el variante del parámetro -y siguiente.

Para los modelos A, B Rev 2 o B+:
```
$ sudo i2cdetect -y 1
```

Para el modelo original B Rev 1:
```
$ sudo i2cdetect -y 0
```

En mi caso tengo el modelo B+ así que ejecuto el siguiente comando:
```
$ sudo i2cdetect -y 1
```

Nos devolvera algo asi:
```
0 1 2 3 4 5 6 7 8 9 a b c d e f
00: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

Conectamos la pantalla LCD con el circuito de expansión. La conexión es muy simple.
Conectamos nuestra pantalla Raspberry pi con PCF8574:
- GND negativo.
- VCC positivo 5V.
- SDA.
- SCL.

Una vez conectada nuestra pantalla con el circuito PCF8574, volvemos a ejecutar el comando:
```
$ sudo i2cdetect -y 1
```


```
0 1 2 3 4 5 6 7 8 9 a b c d e f
00: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 3f
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

Con 3f identificamos la dirección para controlar la pantalla.
En la siguiente entrada continuare explicando el uso de la pantalla LCD, con código Python disponible en mi Github.

https://vine.co/v/edhvIb16Bnw