---
layout: post
title: "Instalación manual de librerías en QPython [Android]"
date: 2016-05-15 16:25:06 -0700
comments: true
---

Como todos los años me toca consultar los decimos de la lotería de navidad de mi familia, así que para ahorrar tiempo, cree un script en Python para consultar los boletos.

Ya que normalmente me suelen asaltar en cualquier lugar para que se los consulte, decidí poder ejecutar el script en mi teléfono para hacerlo portable. La aplicación que utilice es <strong>QPython</strong>. El problema que me encontré con esta aplicación fue que no disponía de librerías esenciales como <strong>request</strong>. Para solucionar este error decidí instalar esta libreria de forma manual ya que no encontré otra forma de instalarlo en la aplicación.

### Instalar request de forma manual en QPython para Android:

Lo primero que tenemos que hacer es encontrar la ubicación donde tiene las librerías QPython, en mi caso fue: `/mnt/sdcard/com.hipipal.qpyplus/lib/python2.7/site-packages/`

Lo siguiente fue **descargar request** de (https://pypi.python.org/pypi/requests/) como requests-2.9.1.tar.gz.

Por ultimo solo tenemos que **copiar la carpeta request** que se encuentra dentro del paquete requests-2.9.1.tar.gz a la ruta anterior, /mnt/sdcard/com.hipipal.qpyplus/lib/python2.7/site-packages/

Una vez echo esto ya tendremos funcionando nuestra librería de request.

### BONUS:

El script en cuestión de la lotería de navidad del 2015:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# https://github.com/Tedezed

import requests

list_numeros = {'89650', '39204', '49038'}

for numero in list_numeros:
respuesta = requests.get('http://api.elpais.com/ws/LoteriaNavidadPremiados', params={'n': numero })
exec respuesta.text
if busqueda['error'] == 0:
print 'Numero ' + str(busqueda['numero']) + ' premio: ' + str(busqueda['premio'])
else:
print 'ERROR en numero ' + str(numero)

if busqueda['status'] == 0:
print 'El sorteo no ha comenzado aún.'
if busqueda['status'] == 1:
print 'El sorteo ha empezado.'
if busqueda['status'] == 2:
print 'El sorteo ha terminado, lista no segura.'
if busqueda['status'] == 3:
print 'El sorteo ha terminado y existe una lista oficial en PDF.'
if busqueda['status'] == 4:
print 'El sorteo ha terminado y la lista de números y premios está basada en la oficial.'
```

La API utilizada es api.elpais.com la cual nos devuélvele un JSON en formato erroneo:
```busqueda={"numero":89765,"premio":0,"timestamp":1450785981,"status":1,"error":0}```

Que debería ser de este modo:
```{"busqueda" : {"numero":89765,"premio":0,"timestamp":1450785981,"status":1,"error":0}}```

Un saludo!