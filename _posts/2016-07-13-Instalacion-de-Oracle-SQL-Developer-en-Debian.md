---
layout: post
title: "Instalación de Oracle SQL Developer en Debian"
date: 2016-05-13 16:25:06 -0700
comments: true
---

En este post seguiremos con la instalación de herramientas de Oracle en Debian.

Para comenzar, **después de instalar SQL Plus en Debian** descargamos el siguiente paquete rpm de la [web de Oracle](http://www.oracle.com/technetwork/developer-tools/sql-developer/).

Instalamos el paquete rpm con Alien: ```alien -i sqldeveloper-*.rpm```

Una vez que termine la instalación, tendremos que instalar JDK como dependencia, en este caso instalare el paquete privativo, pero también funciona bien con el paquete libre.

Instalamos JDK:

Añadimos repositorios, aunque sea de ubuntu, funciona correctamente en Debian:
```echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee /etc/apt/sources.list.d/webupd8team-java.list```
```echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list```

Clave para el repositorio: ```apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886```

Actualizamos la lista de repositorios: ```apt-get update```

Instalamos JDK8 en Debian:
```apt-get install oracle-java8-installer oracle-java8-set-default```

Editamos el siguiente archivo **/home/user_x/.sqldeveloper/4.1.0/product.conf** y especificamos la ruta de nuestra instalación de Java JDK, escribiendo la siguiente linea: ```SetJavaHome /usr/lib/jvm/java-8-oracle/```

Para entrar en SQL Developer, ejecutamos sqldeveloper. Opcionalmente podemos añadir un lanzador a GNOME.

Con esto termina la instalación. Cualquier duda pregunten por comentario.

Un saludo!