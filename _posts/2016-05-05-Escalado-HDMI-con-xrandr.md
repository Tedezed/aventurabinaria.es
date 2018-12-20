---
layout: post
title: "Escalado HDMI con xrandr"
date: 2016-05-05 16:25:06 -0700
comments: true
---

En esta entrada ofrezco una solución para los que tenemos pantallas de televisión como monitores conectados por HDMI, en principio según con que resolución la configuremos podremos tener el problema de que la imagen se muestra con demasiado zoom. Para solucionar esto haremos un escalado con xrandr.

En primer lugar con xrandr vemos las salidas de vídeo de nuestra gráfica en mi caso tengo una ATI Radeon R9 270X de Gigabite y este es el resultado.

	~$ xrandr
	Screen 0: minimum 320 x 200, current 1920 x 1080, maximum 16384 x 16384
	DisplayPort-0 disconnected (normal left inverted right x axis y axis)
	HDMI-0 connected primary 1920x1080+0+0 (normal left inverted right x axis y axis) 478mm x 268mm
	   1366x768      59.79 +
	   1920x1080     60.00    50.00    59.94* 
	   1920x1080i    60.00    50.00    59.94  
	   1280x1024     60.02  
	   1280x720      60.00    50.00    59.94  
	   1440x576i     50.00  
	   1024x768      60.00  
	   1440x480i     60.00    59.94  
	   800x600       60.32  
	   720x576       50.00  
	   720x480       60.00    64.37    59.94  
	   640x480       60.00    59.94    59.94  
	   720x400       70.08  
	DVI-0 disconnected (normal left inverted right x axis y axis)
	DVI-1 disconnected (normal left inverted right x axis y axis)

El driver que utilizo actualmente es el controlador libre de Debian. <a href="https://wiki.debian.org/AtiHowTo">AtiHowTo</a>

En segundo lugar habilitamos underscan en nuestra salida HDMI-0

	xrandr --output HDMI-0 --set underscan on

Por ultimo configuramos el borde horizontal hborder y el vertical vborder

	xrandr --output HDMI-0 --set "underscan hborder" 41 --set "underscan vborder" 25

Por ultimo para guardar los cambios podemos añadir los comandos al Init de Gnome `/etc/gdm3/Init/Default` justo debajo de 

	PATH="/usr/bin:$PATH"
	OLD_IFS=$IFS

Quedando así

	PATH="/usr/bin:$PATH"
	OLD_IFS=$IFS

	xrandr --output HDMI-0 --set underscan on
	xrandr --output HDMI-0 --set "underscan hborder" 41 --set "underscan vborder" 25

Con esto ya tendremos configurado el escalado HDMI.