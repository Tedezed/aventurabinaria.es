---
layout: post
title: "Cifrado de volúmenes lógicos (LVM) con Cryptsetup (dm-crypt)"
date: 2016-07-04 16:25:06 -0700
comments: true
---

Para el cifrado de volúmenes lógicos utilizaremos el sistema de cifrado dm-crypt con:
Volumen lógico cifrado asociado al punto de montaje en la ruta /opt2.
Volumen lógico se montara automáticamente al arrancar la máquina.

[su_spoiler title="Configuración:" style="default" open="yes"]
Instalamos cryptsetup con:
<div class="shell">
<p style="padding-left: 20px;"><code>apt-get install cryptsetup
</code></p>

</div>
Creamos un volumen lógico en el grupo de volúmenes LVD-Debian con el nombre cifrado y de 1G:
<div class="shell">
<p style="padding-left: 20px;"><code>lvcreate -n cifrado -L 1G LVD-Debian
</code></p>

</div>
Encriptamos el volumen con cryptsetup, este comando nos pedirá la contraseña para cifrar:
<div class="shell">
<p style="padding-left: 20px;"><code>cryptsetup --verify-passphrase --verbose create cifrado /dev/LVD-Debian/cifrado
</code></p>

</div>
Damos formato XFS al volumen:
<div class="shell">
<p style="padding-left: 20px;"><code>mkfs.xfs /dev/mapper/cifrado
</code></p>

</div>
Editamos: nano /etc/crypttab
<div class="shell">
<p style="padding-left: 20px;"><code>cifrado /dev/LVD-Debian/cifrado none
</code></p>

</div>
Editamos con nano /etc/fstab para que se monte al inicio como opt2:
<div class="shell">
<p style="padding-left: 20px;"><code>/dev/mapper/cifrado /opt2 xfs defaults,noatime,nodiratime 1 2
</code></p>

</div>
Reiniciamos para ver el funcionamiento del montaje automático del volumen cifrado, este sera el mensaje que nos aparecerá:
<a href="http://www.aventurabinaria.es/wp-content/uploads/2015/11/IMG_20151103_d.jpg"><img class="alignnone size-medium wp-image-366" src="http://www.aventurabinaria.es/wp-content/uploads/2015/11/IMG_20151103_d-744x376.jpg" alt="lvm encriptado debian" width="744" height="376" /></a>
Introducimos la contraseña para descifrar.
[/su_spoiler]

[su_spoiler title="Eliminar volúmenes:" style="default" open="yes"]
Para eliminar el volumen; eliminamos las lineas añadidas en fstab y cryptab. Reiniciamos.
Al arrancar de nuevo ejecutamos como root los siguientes comandos:
<div class="shell">
<p style="padding-left: 20px;"><code>cryptsetup remove cifrado
lvremove /dev/LVD-Debian/cifrado
</code></p>

</div>
[/su_spoiler]
Con esto ya tendríamos terminado nuestro entorno de volúmenes cifrados.