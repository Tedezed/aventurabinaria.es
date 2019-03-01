---
layout: post
title: "Migración de maquina virtual KVM en vivo"
date: 2016-05-17 16:25:06 -0700
comments: true
---


En esta entrada podremos ver la migración de una maquina en [kvm](http://www.linux-kvm.org) (VM-1) con dos volúmenes, a otra maquina (VM-2) redimensionando de camino sus volúmenes lógicos y aumentando su capacidad en vivo.

La maquina VM-1 contara de dos volúmenes; el raíz donde se encuentra la instalación de Debian y otro con Postgres. A continuación expongo el orden de la migración:

#### Orden de migración:

- Snapshot de de disco raíz VM-1.
- Nuevo volumen de 2GB.
- Con dd pasamos el snapshot de raiz de VM-1 al volumen de 2GB (VM-2-raiz).
- Creamos nueva maquina con VM-2-raiz.
- Paramos Postgres.
- Desmontamos volumen Postgres.
- Redimensionamos volumen Postgres.
- Montamos en VM-2 el volumen de Postgres.
- Arrancamos Postgres en VM-2.
- Cambio de regla DNAT.

### Instalación y configuración de VM-1
En primer lugar explicare la instalación de la maquina VM-1.

**En anfitrión:**

Creamos dos volúmenes lógicos:
```lvcreate -n VM-1-raiz -L 1G LVD-Debian```
```lvcreate -n postgres-lvm -L 1G GLVM```

Copiamos configuración previa:
```cp /etc/libvirt/qemu/Debian-Test.xml /etc/libvirt/qemu/VM-1.xml```

Buscar y remplazar (Cambio de nombre, eliminar uuid y mac):
```sed -i 's/Debian-Test/VM-1/' /etc/libvirt/qemu/VM-1.xml; sed -i /uuid/d /etc/libvirt/qemu/VM-1.xml; sed -i '/mac address/d' /etc/libvirt/qemu/VM-1.xml```

Añadimos los volúmenes:
```nano /etc/libvirt/qemu/VM-1.xml```

Arrancamos la red:
```virsh net-start default```

Definir VM-1:
```virsh define /etc/libvirt/qemu/VM-1.xml```

Arrancar VM-1:
```virsh start VM-1```

**En VM-1:**

Instalamos Debian, en el volumen raíz.

Damos formato xfs al volumen:
```mkfs.xfs /dev/vdb```

Añadimos el volumen a /etc/fstab
```/dev/vdb /var/lib/postgresql xfs defaults 0 1```

Reiniciamos la maquina:
```reboot```

Instalamos Postgres:
```apt-get install postgresql-9.4```
Configuración de Postgres:

Entramos en el directorio main de Postgres 9.4:
```nano /etc/postgresql/9.4/main/postgresql.conf```

Editamos el archivo de configuración postgresql.conf:
Buscamos la siguiente linea:
```#listen_addresses = 'localhost'```

Ponemos especificar la IP remota:
```listen_addresses = '172.22.7.88'```

También la podemos cambiar por '\*' para que se puedan conectar todas las IP:
```listen_addresses = '*'```

Buscamos al siguiente linea:
```#password_encryption = on```

Habilitamos el cifrado de la contraseña:
```password_encryption = on```

Editamos el siguiente archivo:

```nano /etc/postgresql/9.4/main/pg_hba.conf```

Con este archivo podemos configurar las relaciones de confianzas de host y redes.
```host all all 127.0.01/32 md5```

También podemos dar acceso a todas las redes con:
```host all all 0.0.0.0 0.0.0.0 md5```

Reiniciamos el servicio con:
```service postgresql restart```

Creamos usuario morrigan para realizar una prueba:
```
CREATE USER morrigan PASSWORD 'morrigan';
ALTER ROLE morrigan CREATEDB;
\q
psql -h localhost -d postgres -U morrigan
CREATE DATABASE programas;
\q
```

**En anfitrión:**

Modificamos iptables:
Limpiamos FORDWARD:
```iptables -F FORWARD```

**DNAT:**
```iptables -t nat -A PREROUTING -p tcp --dport 5432 -i wlan0 -j DNAT --to 192.168.122.232```

Desde otro equipo probamos la configuración anterior:
```psql -h IP-anfitrion -U morrigan```

### Migración de VM-1 a VM-2

**En el anfitrión:**

Creamos un snapshot a un volumen logico:
```lvcreate --snapshot -L1G -n VM-1-raiz-snapshot LVD-Debian/VM-1-raiz```

Creamos un volumen final para VM-2 con 2GB:
```lvcreate -n VM-2-raiz -L 2G LVD-Debian```

Volcamos de VM-1-raiz-snapshot a VM-2-raiz
```dd if=/dev/LVD-Debian/VM-1-raiz-snapshot of=/dev/LVD-Debian/VM-2-raiz```
Creamos una nueva maquina VM-2:
Copiamos configuración previa:
```cp /etc/libvirt/qemu/VM-1.xml /etc/libvirt/qemu/VM-2.xml```

Buscar y remplazar:
```sed -i 's/VM-1/VM-2/' /etc/libvirt/qemu/VM-2.xml; sed -i /uuid/d /etc/libvirt/qemu/VM-2.xml; sed -i '/mac address/d' /etc/libvirt/qemu/VM-2.xml; sed -i 's/524288/1048576/' /etc/libvirt/qemu/VM-2.xml```

Añadimos el volumen anterior:
```nano /etc/libvirt/qemu/VM-2.xml```

Definimos el nuevo server VM-2:
```virsh define /etc/libvirt/qemu/VM-2.xml```

Arrancamos el nuevo server VM-2:
```virsh start VM-2```

**En VM-2:**

Modificamos la tabla de particiones:
```fdisk /dev/vda```

Eliminamos la partición con: d
Creamos una nueva con: n
Abarcamos el disco entero.
Finalmente escribimos los cambios: w

Extendemos con xfs_growfs la partición xfs:
```xfs_growfs -d /dev/vda1```

Comprobamos el resultado con:
```
root@debian:/home/debian# df -h
S.ficheros Tamaño Usados Disp Uso% Montado en
/dev/vda1 2,0G 821M 1,2G 41% /
```

Con esto, ya tendríamos montado el disco raíz de la maquina en producción VM-1 en VM-2, mientras VM-1 sigue prestando servicio.

**En VM-1:**

Paramos postgres:
```service postgresql stop```

Desmontamos el directorio:
```umount /var/lib/postgresql```

**En anfitrión:**

Desacoplamos el disco vdb de VM-1:
```virsh detach-disk VM-1 vdb```

Redimensionamos el volumen:
```lvresize -L +1GB /dev/LVD-Debian/postgres-lvm```

Acoplamos el volumen de VM-1 en VM-2:
```virsh attach-disk VM-2 /dev/LVD-Debian/postgres-lvm sdb```

**En VM-2:**

Cambiamos vdb por sda nano /etc/fstab
```/dev/sda /var/lib/postgresql xfs defaults 0 1```

Montamos con:
```mount -a```

Redimensionamos xfs:
```xfs_growfs -d /dev/sda```

Iniciamos Postgresql:
```/etc/init.d/postgresql start```

Por ultimo modificamos iptables para cambiar el trafico del puerto 5432:
Eliminamos la regla anterior:
```iptables -t nat -D PREROUTING -p tcp --dport 5432 -i wlan0 -j DNAT --to 192.168.122.232```

Añadimos la regla nueva:
```iptables -t nat -A PREROUTING -p tcp --dport 5432 -i wlan0 -j DNAT --to 192.168.122.73```

Con esto terminamos las migración.