---
layout: post
title: "Instalar cliente Oralce SQL Plus en Debian"
date: 2016-05-16 16:25:06 -0700
comments: true
---

Para instalar SQL PLus de Oracle en Debian de una forma fácil y rápida, tendremos que instalar los siguientes paquetes necesarios en el sistema: ```apt-get install alien rpm libaio1```

Una vez instalados los paquetes anteriores, vamos la [pagina de descargas de Oracle](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html) y descargamos los siguiente paquetes:
```
oracle-instantclient11.2-basic-11.2.0.2.0.x86_64.rpm
oracle-instantclient11.2-sqlplus-11.2.0.2.0.x86_64.rpm
oracle-instantclient11.2-devel-11.2.0.2.0.x86_64.rpm
```

Instalamos los paquetes rpm en Debian, esto es posible gracias a Alien, que nos convertirá los paquetes .rpm en paquetes .deb. EL comando para la instalación de los paquetes con alien, es el siguiente:
```
alien -i oracle-instantclient*-basic*.rpm
alien -i oracle-instantclient*-sqlplus*.rpm
alien -i oracle-instantclient*-devel*.rpm
```

Para que funcione correctamente debemos especificar la ruta de Oracle basic en el archivo **/etc/ld.so.conf.d/oracle.conf** en mi caso es la siguiente:
```/usr/lib/oracle/11.2/client64/lib/```

Posteriormente ejecutamos **ldconfig** para actualizar el vinculo anterior.

Para ejecutar SQL Plus escribimos: ```sqlplus64```


Nos conectamos a nuestro servidor de base de datos Oralce con: ```sqlplus64 usuario/contraseña@IP:PUERTO/Servicio```


### Historial en SQLPlus
Si queremos tener un historial de los comandos introducidos en sqlplus64 tendremos que instalar rlwrap:
```apt-get install rlwrap```

Con rlwrap podremos guardar un historial de los comandos para ejecutarlos seria de la siguiente forma:
```rlwrap sqlplus64 usuario/contraseña@IP:PUERTO/Servicio```

Para dejarlo correctamente, podemos añadir un alias en nuestro .bash_aliases de la siguiente forma:
```nano ~/.bash_aliases```

Dentro introducimos lo siguiente:
```alias sqlplus='rlwrap sqlplus64'```

A la hora de ejecutarlo introducimos el nuevo alias, para que funcione el alias, tenemos que cerrar y abrir de nuevo el terminal:
```sqlplus usuario/contraseña@IP:PUERTO/Servicio```


Con esto ya estaría funcionando correctamente SQL Plus en Debian, para poder conectarnos remotamente a nuestro servidor de base de datos Oracle. Si tenéis cualquier duda o problema, comentarla.

Un cordial saludo!