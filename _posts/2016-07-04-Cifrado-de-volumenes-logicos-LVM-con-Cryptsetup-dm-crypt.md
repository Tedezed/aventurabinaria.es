---
layout: post
title: "Cifrado de volúmenes lógicos (LVM) con Cryptsetup (dm-crypt)"
date: 2016-07-04 16:25:06 -0700
comments: true
---

Para el cifrado de volúmenes lógicos utilizaremos el sistema de cifrado dm-crypt con:
Volumen lógico cifrado asociado al punto de montaje en la ruta /opt2.
Volumen lógico se montara automáticamente al arrancar la máquina.

Instalamos cryptsetup con:
```
apt-get install cryptsetup
```

Creamos un volumen lógico en el grupo de volúmenes LVD-Debian con el nombre cifrado y de 1G:
```
lvcreate -n cifrado -L 1G LVD-Debian
```

Encriptamos el volumen con cryptsetup, este comando nos pedirá la contraseña para cifrar:

```
cryptsetup --verify-passphrase --verbose create cifrado /dev/LVD-Debian/cifrado
```

Damos formato XFS al volumen:

```
mkfs.xfs /dev/mapper/cifrado
```

Editamos: nano /etc/crypttab

```
cifrado /dev/LVD-Debian/cifrado none
```

Editamos con nano /etc/fstab para que se monte al inicio como opt2:

```
/dev/mapper/cifrado /opt2 xfs defaults,noatime,nodiratime 1 2
```

Reiniciamos para ver el funcionamiento del montaje automático del volumen cifrado.

Introducimos la contraseña para descifrar.

Para eliminar el volumen; eliminamos las lineas añadidas en fstab y cryptab. Reiniciamos.
Al arrancar de nuevo ejecutamos como root los siguientes comandos:
```
cryptsetup remove cifrado
lvremove /dev/LVD-Debian/cifrado
```

Con esto ya tendríamos terminado nuestro entorno de volúmenes cifrados.