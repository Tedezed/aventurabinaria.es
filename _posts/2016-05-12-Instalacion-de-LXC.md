---
layout: post
title: "Instalación de LXC"
date: 2016-05-12 16:25:06 -0700
comments: true
---

En esta entrada trataremos la instalación de LXC en Debian Jessie, junto con; comandos básicos y creación de contenedores. Esta instalación se realizara en un sistema auxiliar en un volumen lógico, dado que en el ordenador donde lo montare sera en mi ordenador personal.

### Información general de LXC

LXC (Linux Containers) es un entorno de virtualización a nivel de sistema operativo para la ejecución de múltiples sistemas Linux aislados en contenedores en un único host anfitrión con kernel Linux.

- Virtualización a nivel de sistema operativo.
- Licencia: GNU-GPL v2.
- Escrito en: C, python3, shell, lua.
- Desarrollo activo.
- Sistemas soportados: sistemas con nucleo Linux.
- Arquitecturas: x86, x86-64, IA-64, PowerPC, SPARC, Itanium, ARM.
- Versión estable desde el 12/05/2013
- Web: linuxcontainers.org

Presentación de LXC: (http://tedezed.github.io/Presentation/lxc/)

### Instalación y configuración de LXC
Instalación de paquetes: ```apt-get install lxc bridge-utils libvirt-bin debootstrap```

Debemos comprobar la configuración del núcleo, vemos si todas las características están activas.

Creamos un nuevo contenedor con: ```lxc-create -n contenedor1 -t debian```

Una vez creada la máquina, nos dirá que la contraseña de root es 'root' y nos aconseja que la cambiemos.
```Root password is 'J90Fki5w', please change !```


También podemos crear un contenedor con un volumen para copia de seguridad:
Si ejecutamos directamente el comando para crear el contenedor: ```lxc-create -t debian -n contenedor_lv -B lvm --vgname lxc --lvname lv_debian --fssize 1G --fstype xfs```

Nos devolverá el siguiente error:
```
File descriptor 3 (/var/lib/lxc/contenedor_lv/partial) leaked on lvcreate invocation. Parent PID 4696: lxc-create
<strong>Volume group "lxc" not found</strong>
lxc_container: Error creating new lvm blockdev /dev/lxc/lv_debian size 1073741824 bytes
lxc_container: Failed to create backing store type lvm
lxc_container: Error creating backing store type lvm for contenedor_lv
lxc_container: Error creating container contenedor_lv
```

Para resolver este error creamos un dispositivo de imagen donde crearemos un volumen utilizable desde lxc.
```truncate -s 5G lvm.img```
```losetup -f --show lvm.img```

Anotamos la respuesta del comando, en mi caso /dev/loop0
Con esto ultimo resuelto, creamos el grupo de volúmenes por defecto de lxc:

Elegimos el volumen anterior con:
```pvcreate /dev/loop0```

Damos como nombre lxc: ```vgcreate lxc /dev/loop0```

Creamos un volumen lógico y dejamos espacio para los metadatos: ```lvcreate -l 95%FREE --type thin-pool --thinpool lxc lxc```

Ahora si ejecutamos el comando para crear el contenedor con volumen lógico.
```lxc-create -t debian -n contenedor_lv -B lvm --vgname lxc --lvname lv_debian --fssize 1G --fstype xfs```

```
-B -Tipo de volumen para copia de seguridad: 'dir', 'lvm', 'loop', 'btrfs', 'zfs' o 'best'.
--vgname -Nombre del grupo de volúmenes.
--lvname -Nombre del volumen lógico.
--fssize -Capacidad del volumen lógico.
--fstype -Formato del volumen.
```

También nos retornara la contraseña de root:
```Root password is '9aQC1I1L', please change !```

Configurar bridge en un contenedor:
(https://wiki.debian.org/LXC/SimpleBridge)

Editamos el fichero de red de la máquina anfitriona.
```nano /etc/network/interfaces```

Ponemos las siguientes ordenes suponiendo que nuestra tarjeta de red es la eth0:
```
[enlighter lang="python"]auto eth0
iface eth0 inet dhcp
auto br0
iface br0 inet dhcp
bridge_ports eth0
bridge_fd 0
bridge_maxwait 0
```

Desactivamos Network Manager, este puede que nos de problemas a la hora de utilizar esta configuración:
```/etc/init.d/network-manager stop```

Reiniciamos la red:
```/etc/init.d/networking restart```

Podemos configurar la red de dos formas:
- Configurando el contenedor nano /var/lib/lxc/contenedor1/config
- Configurando la configuración general nano /etc/lxc/default.conf

En ambos casos tendremos que añadir:
```
# Network
lxc.network.type = veth
lxc.network.flags = up
lxc.network.link = br0
```

Iniciamos el contenedor para testear la red: ```lxc-start -n contenedor1 -d```

Entramos como root: ```lxc-console -n contenedor1```

Miramos la ip de la maquina:
```
root@contenedor1:~# ip addr | grep eth0
4: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
inet 192.168.0.6/24 brd 192.168.0.255 scope global eth0
```

Podemos comprobar la conexión con ping, en mi caso no viene instalado en la maquina así que instalamos el paquete:
```
apt-get update
apt-get upgrade
apt-get install iputils-ping
```

Hacemos ping:
```
root@contenedor1:~# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=51 time=47.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=51 time=56.4 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 47.514/51.967/56.421/4.459 ms
```

Para iniciar el contenedor: ```lxc-start -n contenedor1 -d```

Conectar al contenedor anteriormente iniciado: ```lxc-console -n contenedor1```

Para salir pulsamos CTRL+a y al soltar q.

Para ver los contenedores creados: ```lxc-list```

Parar contenedor: ```lxc-stop -n contenedor1```

Apagar el contenedor: ```lxc-halt -n contenedor1```

Borrar contenedor: ```lxc-destroy -n contenedor1```

Si queremos saber más información sobre un contenedor como el pid: ```lxc-info -n contenedor1```

Clonar contenedor: ```lxc-clone -o contenedor1 -n contenedor2```

Auto inicio de contenedores al arrancar el anfitrión:
Editamos con `nano /var/lib/lxc/contenedor1/config` y añadimos: ```lxc.start.auto = 1```

Ejecutar comandos en un contenedor: ```lxc-attach -n contenedor1 -- ip addr```
