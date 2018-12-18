---
layout: post
title: "CMS Ximdex de Vagrant a OpenShift"
date: 2016-07-06 16:25:06 -0700
comments: true
---

<strong>Desplegando Ximdex en un entorno de desarrollo para después ser montado en un entorno de producción.</strong>

<a href="http://aventurabinaria-tedgoo.rhcloud.com/wp-content/uploads/2015/08/vagrant.jpg"><img class="alignnone size-full wp-image-26" src="http://aventurabinaria-tedgoo.rhcloud.com/wp-content/uploads/2015/08/vagrant.jpg" alt="vagrant" width="700" height="534" /></a>

Configuramos entorno de producción, OpenShift:
<ul>
	<li>Seleccionamos PHP 5.4</li>
	<li>Añadimos cartridge de MySQL 5.5</li>
</ul>
En este caso el entorno de desarrollo sera una maquina virtual en vagrant:
Partiendo de que ya tenemos configurado vagrant, creamos el directorio de la maquina:
<pre>mkdir lamp_openshift</pre>
Ejecutamos <code>vagrant init</code> para crear el archivo Vagrantfile.
Modificamos, configuramos una maquina vagrant con una interfaz privada de red para poder conectarnos, memoria y cpu a usar:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
#Box de Vagrant
config.vm.box = "precise64"
#Red privada para hacer pruebas de CMS
config.vm.network "private_network", ip: "192.168.40.50"
config.vm.provider :virtualbox do |vb|
vb.customize ["modifyvm", :id, "--memory", "1024"]
vb.customize ["modifyvm", :id, "--cpus" , 2]
end
end
```

Iniciamos la maquina con:
<pre>vagrant up</pre>
Entramos en la maquina con:
<pre>vagrant ssh</pre>
Una vez dentro ejecutamos:
<pre>sudo apt-get update</pre>
Instalamos git:
<pre>sudo apt-get install git</pre>
Entramos en el directorio .ssh y generamos una clave publica/privada, damos un nombre, por ejemplo "vagrant":
<pre>ssh-keygen -t rsa -C "tu correo del PaaS"</pre>
Esto nos generara dos archivos una clave privada "vagrant" y una publica "vagrant.pub", el contenido de esta ultima tendremos que facilitarla a nuestro entorno de producción para poder conectarnos por ssh.
Cambia los permisos de clave privada a 400 o 601 si no lo tiene por defecto.
<pre>ssh-agent /bin/bash
ssh-add "clave privada-fichero"</pre>
Subimos la clave publica al PaaS.

Instalamos en el entorno de desarrollo las librerias nesesarias:
<pre>apt-get install apache2 mysql-server mysql-client php5 php5-mysql php5-common php5-cli php5-gd php-pear php5-mcrypt php5-enchant php5-curl php5-xsl</pre>
<strong>Para cualquier duda de los requisitos, visita:</strong>
<a title="http://www.ximdex.org/documentacion/requisitos_es.html" href="http://www.ximdex.org/documentacion/requisitos_es.html">http://www.ximdex.org/documentacion/requisitos_es.html</a>

Descargamos Ximdex:
<pre>~$ wget https://codeload.github.com/XIMDEX/ximdex/tar.gz/v3.5</pre>
Descomprimimos Ximdex:
<pre>tar zxf v3.5</pre>
Esto creara una directorio llamado ximdex-3.5 y Clonamos el directorio del entorno de producción:
<pre>git clone ssh://************************@ximdex-tedezed.rhcloud.com/~/git/ximdex.git/</pre>
Esto creara una directorio llamado ximdex, que es nuestro repositorio. Entramos dentro de ximdex-3.5 y ejecutamos:
<pre>mv * /var/www/ximdex</pre>
Esto moverá Ximdex al repositorio del PaaS en local, despues cambiamos el propietario para apache2:
<pre>sudo chown -R www-data:www-data ximdex/</pre>
Reiniciamos apache2:
<pre>/etc/init.d/apache2 restart</pre>
Es importante subir aquí Ximdex al PaaS ya que el "CHECK CONFIGURATION" bajo mi experiencia solo se ejecuta la primera vez, después arranca siempre donde lo dejamos por ultima vez la configuración, por lo cual antes de seguir subimos ximdex.
<pre>git add *
git commit -m "tu comentario"
git push</pre>
Una vez subido probamos que todo este en orden en Openshift, si hace falta alguna librería solo tendremos que instalarla. Dado que el despliegue se hace desde el entorno de desarrollo hacia el entorno de producción en el servidor no configuraremos nada, todo se configurara en el entorno de desarrollo y después una vez que funcione correctamente en local, se subirá.

Entramos en MySQL en local:
<pre>mysql -u "Nuestro usuario Root" -p</pre>
CMS Ximdex nos pedirá la cuenta de root de mysql y el se encargaría de todo, aunque es lo mas cómodo no es lo correcto ya que podríamos comprometer la seguridad del sistema, creamos un usuario con su db y le asignamos permisos.

Creamos la base de datos:
<pre>create database TU_DB;
use TU_DB;</pre>
Creamos el usuario:
<pre>create user TU_USUARIO;</pre>
Asignamos permisos:
<pre>GRANT ALL ON nombre_db.* TO usuario_db IDENTIFIED BY 'contraseña';</pre>
Salimos de MySQL:
<pre>exit</pre>
Entramos desde el navegador a ximdex para configurarlo:
http://192.168.40.50/ximdex/

Aquí lo único que tendremos que hacer es configurar los parámetros de la db, como usuario, nombre db y contraseña, creamos un nuevo usuario con el que después nos logueamos.
Instalamos los modulos Xtags, Xtour, Xnews. Esto puede tardar unos minutos, ya que nuestra maquina vagrant no dispone de mucha potencia.

Si todo a salido bien nos mostrara el siguiente mensaje:
<pre>Installation finished!////////////////////////////////////////////////////////////
You've succesfully installed Ximdex CMS on your server.
Log in and discover a different way to manage your content and data.

Enable decoupled dynamic semantic publishing by adding these lines to your root crontab:

* * * * * (php /var/www/ximdex/modules/ximNEWS/actions/generatecolector/automatic.php) &gt;&gt; /var/www/ximdex/logs/automatic.log 2&gt;&amp;1

* * * * * (php /var/www/ximdex/modules/ximSYNC/scripts/scheduler/scheduler.php) &gt;&gt; /var/www/ximdex/logs/scheduler.log 2&gt;&amp;1

</pre>
Añadimos las dos lineas anteriores al demonio contrab de maquina vagrant y Openshift, estas se encargaran de ejecutar la publicación.
Comentamos disable_functions en /etc/php5/cli/php.ini y /etc/php5/apache2/php.ini

Nos logueamos en Ximdex con el usuario anteriormente creado y cremos un nuevo proyecto.
En este caso para ver un ejemplo simple, con los temas incluidos nos bastará, asignamos un nombre, canal y un tema.
<strong>
Para cualquier duda en la creación de proyectos vease:</strong>
http://translate.google.com/translate?sl=en&amp;tl=es&amp;u=https://github.com/XIMDEX/ximdex/wiki/Recipes#create-a-new-server-medium

Ya tenemos Ximdex funcionando en local, es hora de importar el contenido configurado de nuestro entorno de desarrollo, cms ximdex y db mysql.
En la maquina vagrant, exportamos la db de ximdex en MySql al archivo db_ximdex.sql.
<pre>mysqldump -u usuario_db -p nombre_db &gt; db_ximdex.sql</pre>
Estructura de la DB MySql:

```
+---------------------------------+
| Tables_in_ximdex |
+---------------------------------+
| Actions |
| ActionsStats |
| Batchs |
| ChannelFrames |
| Channels |
| Config |
| Contexts |
| Dependencies |
| Encodes |
| FastTraverse |
| GraphSerieProperties |
| GraphSerieValues |
| GraphSeries |
| Graphs |
| Groups |
| IsoCodes |
| Languages |
| Links |
| List |
| List_Label |
| Locales |
| Messages |
| Namespaces |
| NoActionsInNode |
| NodeAllowedContents |
| NodeConstructors |
| NodeDefaultContents |
| NodeDependencies |
| NodeEdition |
| NodeFrames |
| NodeNameTranslations |
| NodeProperties |
| NodeRelations |
| NodeSets |
| NodeTypes |
| Nodes |
| NodesToPublish |
| NodetypeModes |
| Permissions |
| PipeCacheTemplates |
| PipeCaches |
| PipeNodeTypes |
| PipeProcess |
| PipeProperties |
| PipePropertyValues |
| PipeStatus |
| PipeTransitions |
| Pipelines |
| PortalVersions |
| Protocols |
| PublishingReport |
| Pumpers |
| RelBulletinXimlet |
| RelColectorList |
| RelFramesPortal |
| RelGroupsNodes |
| RelLinkDescriptions |
| RelNewsArea |
| RelNewsBulletins |
| RelNewsColector |
| RelNodeMetadata |
| RelNodeSetsNode |
| RelNodeSetsUsers |
| RelNodeTypeMetadata |
| RelNodeTypeMimeType |
| RelNodeVersionMetadataVersion |
| RelPvdRole |
| RelRolesActions |
| RelRolesPermissions |
| RelRolesStates |
| RelSectionXimlet |
| RelServersChannels |
| RelServersNodes |
| RelServersStates |
| RelStrDocChannels |
| RelStrdocAsset |
| RelStrdocCss |
| RelStrdocNode |
| RelStrdocScript |
| RelStrdocStructure |
| RelStrdocTemplate |
| RelStrdocXimlet |
| RelTagsDescriptions |
| RelTagsNodes |
| RelTemplateContainer |
| RelUsersGroups |
| RelVersionsLabel |
| RelXimletNode |
| Roles |
| SearchFilters |
| SectionTypes |
| ServerErrorByPumper |
| ServerFrames |
| Servers |
| States |
| StructuredDocuments |
| Synchronizer |
| SynchronizerDependencies |
| SynchronizerDependenciesHistory |
| SynchronizerGroups |
| SynchronizerHistory |
| SynchronizerStats |
| SystemProperties |
| UpdateDb_historic |
| Updater_DiffsApplied |
| Users |
| Versions |
| XimIOExportations |
| XimIONodeTranslations |
| XimNewsAreas |
| XimNewsBulletins |
| XimNewsCache |
| XimNewsColector |
| XimNewsFrameBulletin |
| XimNewsFrameVersion |
| XimNewsList |
| XimNewsNews |
| XimTAGSTags |
+---------------------------------+
```

Realizamos un git push.

Nos conectamos por ssh y para cambiar los parametros de la db MySql del PaaS, para ello en primer lugar importamos la db.

Cambiamos la configuración de la db de ximdex:
<pre>nano /var/www/myximdex/conf/install-params.conf.php</pre>
```
/* DATABASE_PARAMS (do not remove this comment, please) */
$DBHOST = "localhost";
$DBPORT = "3306";
$DBUSER = "XIMDEX_DBUSER";
$DBPASSWD = "XIMDEX_DBPASS";
$DBNAME = "myximdexDB";

$XIMDEX_ROOT_PATH = "/var/www/myximdex";;
```

Tenemos que actualizar la db con los parametros nuevos del PaaS, podemos acerlo de dos oformas:
Importamos la DB:
<pre>cd app-root/repo/
mysql -u usuario_mysql nombre_bbdd -p &lt; ruta_fichero_importación.sql</pre>
<ul>
	<li>Utilizar estas dos lineas dentro de MySql:</li>
</ul>
<pre><code>UPDATE Config SET ConfigValue='http://ximdex-tedezed.rhcloud.com/' WHERE ConfigKEY='UrlRoot';</code></pre>
<pre>UPDATE Config SET ConfigValue='/var/lib/openshift/***************/app-root/repo/' WHERE ConfigKEY='AppRoot';</pre>
<ul>
	<li>Utilizar el siguiente script en python, antes de importar la db:</li>
</ul>

```
import sys
fil = sys.argv[1]
cad1 = raw_input('Cadena a sustituir: ')
cad2 = raw_input('Cadena que introducir: ')
#Read
archi = open(fil,'r')
sql = ''
for linea in archi:
sql = sql + linea
sql = sql.replace(cad1,cad2)
#sql = sql.replace(cad3,cad4)
archi.close()
#Write
archi2 = open(fil,'w')
archi2.write(sql)
archi2.close()
```

A este script le pasamos como parametro db_ximdex.sql, se ejecutaria de la siguiente forma:
<pre>python nombre_script.py db_ximdex.sql
</pre>
192.168.40.50/ximdex
por:
ximdex-tedezed.rhcloud.com

Lo ejecutamos de nuevo y sustituimos:

/var/www/ximdex
por:
/var/lib/openshift/***************/app-root/repo/

Por ultimo importas la db:
<pre>cd app-root/repo/
mysql -u usuario_mysql nombre_bbdd -p &lt; ruta_fichero_importación.sql</pre>
Ya estaría listo el despliegue de Ximdex, las publicaciones por defecto estarían en:
http://URL/data/previos/
Publicación de ejemplo:
http://ximdex-tedezed.rhcloud.com/data/previos/