---
layout: post
title: "Añadir volumen adicional a una instancia OpenStack"
date: 2016-05-04 16:25:06 -0700
comments: true
---

En esta entrada aprenderemos a añadir un volumen a una instancia de OpenStack, el procedimiento sera muy parecido en otras plataformas.

En primer lugar tendremos que crear un volumen para después asociarlo a nuestra maquina virtual. Quedara de la siguiente forma. En mi caso, cree dos volúmenes para dos maquinas virtuales.

<a href="http://www.aventurabinaria.es/wp-content/uploads/2016/05/Captura-de-pantalla-de-2016-05-04-080407.png"><img src="http://www.aventurabinaria.es/wp-content/uploads/2016/05/Captura-de-pantalla-de-2016-05-04-080407-744x260.png" alt="Captura de pantalla de 2016-05-04 08:04:07" width="744" height="260" class="alignnone size-medium wp-image-842" /></a>

Como podemos ver en la imagen anterior, OpenStack al asociar el volumen, nos lo añadió en `/dev/vdb`, para comprobar que tenemos nuestro volumen asociado a la  instancia ejecutamos lo siguiente:

`lsblk`

	NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	vda    253:0    0   8G  0 disk 
	└─vda1 253:1    0   8G  0 part /
	vdb    253:16   0  10G  0 disk 

Ya que tenemos nuestro volumen correctamente asociado, el primer paso es dar formato al mismo:

	mkfs.xfs /dev/vdb

Creamos el directorio donde montaremos el volumen:

	mkdir /shared

Montamos el volumen en el directorio anterior

	mount /dev/vdb /shared

Comprobamos el estado del volumen

`df -h`

	S.ficheros     Tamaño Usados  Disp Uso% Montado en
	/dev/vda1        8,0G   1,5G  6,6G  19% /
	devtmpfs         227M      0  227M   0% /dev
	tmpfs            245M      0  245M   0% /dev/shm
	tmpfs            245M   8,3M  237M   4% /run
	tmpfs            245M      0  245M   0% /sys/fs/cgroup
	tmpfs             49M      0   49M   0% /run/user/1000
	/dev/vdb          10G    33M   10G   1% /shared

Por ultimo para que se monte automáticamente lo añadimos a `/etc/fstab`

`nano /etc/fstab`

	/dev/vdb /shared       xfs     defaults        0       0

Con esto ya tendríamos listo nuestro volumen.