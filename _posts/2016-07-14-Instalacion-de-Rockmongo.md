---
layout: post
title: "Instalación de Rockmongo"
date: 2016-05-14 16:25:06 -0700
comments: true
---

En esta entrada explicare como instalar [Rockmongo](http://rockmongo.com/) en Debian de forma fácil, utilizando repositorios de código de GitHub. Esta herramienta permite realizar labores básicas de administración por medio del navegador. En un principio la configuración posterior de la instalación del mismo debería asegurar solo el acceso desde la red interna o locahost ya que podría comprometer la seguridad de nuestro servidor.

Instalación previa de [Mongodb](https://www.mongodb.org/) desde repositorios oficiales de Debian:
```apt-get install mongodb```

Instalación de phph5, php5-dev, apache y git como dependencias de RockMongo:
```sudo apt-get install apache2 php5 php5-dev git```

Instalación de extensión para PHP de Mongodb:
Descargamos en el directorio tmp ya que este eliminara el contenido temporal posteriormente:
```
cd /tmp/
git clone https://github.com/mongodb/mongo-php-driver-legacy.git
cd mongo-php-driver-legacy
```

Proceso de compilación:
```
phpize
./configure
make all
```
Si la compilación se realizo con éxito, solo faltaría realizar la instalación como root con el siguiente comando:
```# make install```

Editamos y añadimos la siguiente linea, para activar la extensión:
```sudo nano /etc/php5/apache2/php.ini```

Buscar por: ; Dynamic Extensions ```extension=mongo.so```

Instalación de RockMongo desde repositorio GitHub asegurando la ultima versión en desarrollo, pero si queréis una versión oficial final lo recomendable es bajarla desde la web oficial:
```
cd /var/www/html/
git clone https://github.com/iwind/rockmongo.git
```

Es recomendable cambiar la contraseña por defecto de esta herramienta, la cambiar editando el siguiente fichero:
`nano /var/www/html/rockmongo/config.php`
```$MONGO["servers"][$i]["control_users"]["admin"] = "admin";```

Reinicio de Apache2 para cargar los últimos cambios:
```# systemctl restart apache2.service```

Para entrar en Rockmongo entramos desde el navegador a: (http://127.0.0.1/rockmongo/)

Con esto termino mi entrada por ahora.

Un saludo!