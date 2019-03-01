---
layout: post
title: "Instalación de Debian y driver WiFi de Windows [C54RI]"
date: 2016-05-11 16:25:06 -0700
comments: true
---


En esta entrada podréis seguir una instalación de un Debian Jessie 8.2, documentada en un equipo de unos 11 años de edad, compuesto por; un <a href="http://www.cpu-world.com/CPUs/Pentium_4/index.html">CPU Intel Pentium 4 a 3,4 GHz</a>, 2 Gb de RAM DDR1, un disco duro de 500Gb, una tarjeta <a href="http://www.conceptronic.net/es/download_list.php?stype=3&amp;productid=337">PCI Conceptronic C54RI V1</a>, una motherboard <a href="http://www.asrock.com/mb/Intel/775i945GZ/index.la.asp">Asrock 775i945GZ</a>.<a href="http://www.aventurabinaria.es/wp-content/uploads/2015/09/start-here-debian.png"><img class="wp-image-220 alignright" src="http://www.aventurabinaria.es/wp-content/uploads/2015/09/start-here-debian.png" alt="logo_debian" width="143" height="143" /></a>

Esta instalación se llevara acabo utilizando volúmenes lógicos y sistema de archivos <a href="https://es.wikipedia.org/wiki/XFS">XFS</a> con escritorio LXDE.

Para esta instalación utilizaré una versión en DVD de Debian que dispone del mismo en 32 y 64 bits.
Si por desgracia tenemos Windows en nuestro equipo y aun así no queremos eliminarlo, tendremos que redimensionar su volumen desde el gestor de disco del que dispone, con esto dejaremos espacio para la instalación.
Es recomendable tener conexión a Internet durante la instalación para poder utilizar una replica de red.

Cuando llegamos al particionado de discos, debemos seleccionar la opción de particionado manual.

Entramos en el gestor de volumen lógicos o LVM, creamos un grupo de volúmenes, damos un nombre al grupo y seleccionamos el espacio libre.
Entramos en creación de volumen lógicos, damos un nombre y asignamos un tamaño, en mi caso de 30Gb.
Creamos un segundo volumen lógico para área de intercambio con el tamaño del doble de la capacidad de nuestra RAM, en mi caso tengo 2Gb este volumen seria de 4Gb.

Para terminar, entramos en nuestros volúmenes lógicos y definimos como raíz el de tamaño de 30Gb con formato XFS y como intercambio el de tamaño de 4Gb.

Posteriormente podremos seleccionar algunas opciones de instalación, como por ejemplo; el escritorio LXDE que es muy ligero y recomendable para equipos poco potentes, las utilidades del sistema, servidor ssh, etc. Cuando termine la instalación, reiniciara el equipo pasando a la siguiente etapa de la instalación donde hablare de la solución de problemas.

Al terminal la instalación observo que Debian no consigue controlar mi tarjeta <a href="http://www.conceptronic.net/es/download_list.php?stype=3&amp;productid=337">PCI Conceptronic C54RI V1</a>. Al visitar la web del fabricante, podemos observar que no tienen soporte para Linux, solo dan soporte para Windows.

Mi solución para que funcione esta tarjeta en Debian es muy simple, solo necesitamos; el controlador para Windows de 32 bits, ya que nuestro Debian es de 32 bits e instalar el paquete <a href="https://wiki.debian.org/NdisWrapper">Ndiswrapper</a>.

El driver lo e sacado del CD que viene con la tarjeta, en este CD podemos encontrar una carpeta llamada WINXP donde encontraremos un archivo llamado rt61.inf, este archivo lo utilizaremos posteriormente, en su defecto, si no tenemos el CD, también lo podemos sacar del ejecutable con unzip:

```unzip -a ejecutable_driver.exe```


Instalamos ndiswrapper para poder utilizar el archivo rt61.inf:

```
apt-get update
apt-get upgrade
apt-get install linux-headers-$(uname -r|sed 's,[^-]*-[^-]*-,,') ndiswrapper-utils-1.9 wireless-tools
```

Instalamos el driver con:

```
ndiswrapper -i rt61.inf
ndiswrapper -mi
```

Listamos los drivers instalados con: ```ndiswrapper -l```

Si por un casual no nos funcionara o nos hemos equivocado, para eliminar el driver ejecutamos: ```ndiswrapper -e rt61```


Una vez que tenemos el controlador correcto, ejecutamos las lineas siguientes para cargar los módulos:
```
depmod -a
modprobe ndiswrapper
```

Configuramos modprobe para que se cargue ndiswrapper cuando la interface de la tarjeta wireless este activado: ```ndiswrapper -m```

En mi caso el modulo en se cargaba al inicio, así que tenemos que hacer la siguiente configuración: ```nano /etc/modules```

Añadimos la linea: ```ndiswrapper```

Con esto finalizamos la instalación, reiniciamos.
Comprobamos que funciona con:

```
root@debian:/home/debian# ndiswrapper -l
rt61 : driver installed
device (1814:0302) present (alternate driver: rt61pci)
```

A continuación hago un breve listado de los módulos que están funcionando correctamente con la instalación.

### Lspci:

```
root@debian:/home/debian# lspci
00:00.0 Host bridge: Intel Corporation 82865G/PE/P DRAM Controller/Host-Hub Interface (rev 02)
00:02.0 Display controller: Intel Corporation 82865G Integrated Graphics Controller (rev 02)
00:06.0 System peripheral: Intel Corporation 82865G/PE/P Processor to I/O Memory Interface (rev 02)
00:1d.0 USB controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #1 (rev 02)
00:1d.1 USB controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #2 (rev 02)
00:1d.2 USB controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #3 (rev 02)
00:1d.3 USB controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #4 (rev 02)
00:1d.7 USB controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) USB2 EHCI Controller (rev 02)
00:1e.0 PCI bridge: Intel Corporation 82801 PCI Bridge (rev c2)
00:1f.0 ISA bridge: Intel Corporation 82801EB/ER (ICH5/ICH5R) LPC Interface Bridge (rev 02)
00:1f.1 IDE interface: Intel Corporation 82801EB/ER (ICH5/ICH5R) IDE Controller (rev 02)
00:1f.2 IDE interface: Intel Corporation 82801EB (ICH5) SATA Controller (rev 02)
00:1f.3 SMBus: Intel Corporation 82801EB/ER (ICH5/ICH5R) SMBus Controller (rev 02)
00:1f.5 Multimedia audio controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) AC'97 Audio Controller (rev 02)
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] RV280 [Radeon 9200 PRO] (rev 01)
01:01.0 Network controller: Ralink corp. RT2561/RT61 rev B 802.11g
01:05.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)
```

### Lsusb:

```
root@debian:/home/debian# lsusb
Bus 005 Device 003: ID 0aec:3260 Neodio Technologies Corp. 7-in-1 Card Reader
Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 002 Device 002: ID 04fc:0005 Sunplus Technology Co., Ltd USB OpticalWheel Mouse
Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
```

### Teclado:

```
root@debian:/home/debian# dmesg | grep keyboard
[ 1.223372] input: AT Raw Set 2 keyboard as /devices/platform/i8042/serio1/input/input1
```

Ratón:

```
root@debian:/home/debian# lsusb | grep Mouse
Bus 002 Device 002: ID 04fc:0005 Sunplus Technology Co., Ltd USB OpticalWheel Mouse
```

### Gráfica AGP:

```
root@debian:/home/debian# lspci | grep VGA
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] RV280 [Radeon 9200 PRO] (rev 01)
```

Buscamos más información sobre el modulo de la grafica:

```
root@debian:/home/debian# ls /sys/class/drm/controlD64/device/driver/module/drivers/
pci:i915 pci:radeon
```

Este modulo corresponde a la grafica AGP de ATI.

```
root@debian:/home/debian# modinfo radeon
filename: /lib/modules/3.16.0-4-686-pae/kernel/drivers/gpu/drm/radeon/radeon.ko
license: GPL and additional rights
description: ATI Radeon
author: Gareth Hughes, Keith Whitwell, others.
firmware: radeon/R520_cp.bin
firmware: radeon/RS600_cp.bin
```

Este modulo pertenece al la grafica integrada de Intel:

```
root@debian:/home/debian# modinfo i915
filename: /lib/modules/3.16.0-4-686-pae/kernel/drivers/gpu/drm/i915/i915.ko
license: GPL and additional rights
description: Intel Graphics
author: Tungsten Graphics, Inc.
alias: pci:v00008086d000022B3sv*sd*bc03sc*i*
alias: pci:v00008086d000022B2sv*sd*bc03sc*i*
```


### Audio:

```
root@debian:/home/debian# lspci | grep audio
00:1f.5 Multimedia audio controller: Intel Corporation 82801EB/ER (ICH5/ICH5R) AC'97 Audio Controller (rev 02)
```


### WIFI PCI:


```
root@debian:/home/debian# lspci | grep RT61
01:01.0 Network controller: Ralink corp. RT2561/RT61 rev B 802.11g
```

Modulo cargado anteriormente para la PCI WiFi:


```
root@debian:/home/debian# basename `readlink /sys/class/net/wlan0/device/driver/module`
ndiswrapper
```

Información del modulo ndiswrapper:

```
root@debian:/home/debian# modinfo ndiswrapper
filename: /lib/modules/3.16.0-4-686-pae/updates/dkms/ndiswrapper.ko
license: GPL
version: 1.59
description: NDIS wrapper driver
author: ndiswrapper team &lt;ndiswrapper-general@lists.sourceforge.net&gt;
srcversion: C101C962263FFF26465A483
depends: usbcore
vermagic: 3.16.0-4-686-pae SMP mod_unload modversions 686
parm: if_name:Network interface name or template (default: wlan%d) (charp)
parm: proc_uid:The uid of the files created in /proc (default: 0). (int)
parm: proc_gid:The gid of the files created in /proc (default: 0). (int)
parm: debug:debug level (int)
parm: hangcheck_interval:The interval, in seconds, for checking if driver is hung. (default: 0) (int)
parm: utils_version:Compatible version of utils (read only: 1.9) (charp)
```

### PCI:

```
root@debian:/home/debian# lsmod | grep pci
rt61pci 26354 0
rt2x00pci 12472 1 rt61pci
rt2x00mmio 12545 1 rt61pci
rt2x00lib 41387 3 rt61pci,rt2x00pci,rt2x00mmio
eeprom_93cx6 12561 1 rt61pci
mac80211 421532 2 rt2x00lib,rt2x00pci
crc_itu_t 12331 2 udf,rt61pci
ehci_pci 12464 0
```


### Ethernet integrado:

```
root@debian:/home/debian# lspci | grep Ethernet
01:05.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)
```

Informción del CPU:

```
root@debian:/home/debian# cat /proc/cpuinfo
processor : 0
vendor_id : GenuineIntel
cpu family : 15
model : 3
model name : Intel(R) Pentium(R) 4 CPU 3.40GHz
stepping : 4
cpu MHz : 3399.647
cache size : 1024 KB
physical id : 0
siblings : 2
core id : 0
cpu cores : 1
apicid : 0
initial apicid : 0
fdiv_bug : no
f00f_bug : no
coma_bug : no
fpu : yes
fpu_exception : yes
cpuid level : 5
wp : yes
flags : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe constant_tsc pebs bts pni dtes64 monitor ds_cpl cid xtpr
bogomips : 6799.29
clflush size : 64
cache_alignment : 128
address sizes : 36 bits physical, 32 bits virtual
```

Memoria RAM y Swap:

```
root@debian:/home/debian# free -o -m
total used free shared buffers cached
Mem: 2022 838 1184 16 28 380
Swap: 3813 0 3813```

</div>
Información de la distribución:
<div class="shell">

```root@debian:/home/debian# lsb_release -idc
Distributor ID: Debian
Description: Debian GNU/Linux 8.2 (jessie)
Codename: jessie```

</div>
Vista general del hardware:
<div class="shell">

```root@debian:/home/debian# lshw -short
H/W path Device Class Description
==========================================================
system Computer
/0 bus Motherboard
/0/0 memory 2022MiB System memory
/0/1 processor Intel(R) Pentium(R) 4 CPU 3.40GHz
/0/1/0.1 processor Logical CPU
/0/1/0.2 processor Logical CPU
/0/100 bridge 82865G/PE/P DRAM Controller/Host-Hub Interface
/0/100/2 display 82865G Integrated Graphics Controller
/0/100/6 generic 82865G/PE/P Processor to I/O Memory Interface
/0/100/1d bus 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #1
/0/100/1d/1 usb1 bus UHCI Host Controller
/0/100/1d.1 bus 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #2
/0/100/1d.1/1 usb2 bus UHCI Host Controller
/0/100/1d.1/1/2 input USB Optical Wheel Mouse
/0/100/1d.2 bus 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #3
/0/100/1d.2/1 usb4 bus UHCI Host Controller
/0/100/1d.3 bus 82801EB/ER (ICH5/ICH5R) USB UHCI Controller #4
/0/100/1d.3/1 usb5 bus UHCI Host Controller
/0/100/1d.7 bus 82801EB/ER (ICH5/ICH5R) USB2 EHCI Controller
/0/100/1d.7/1 usb3 bus EHCI Host Controller
/0/100/1d.7/1/6 scsi4 storage USB Storage Device
/0/100/1d.7/1/6/0.0.0 /dev/sdb disk SCSI Disk
/0/100/1d.7/1/6/0.0.1 /dev/sdc disk SCSI Disk
/0/100/1d.7/1/6/0.0.2 /dev/sdd disk SCSI Disk
/0/100/1d.7/1/6/0.0.3 /dev/sde disk SCSI Disk
/0/100/1e bridge 82801 PCI Bridge
/0/100/1e/0 display RV280 [Radeon 9200 PRO]
/0/100/1e/1 wlan0 network RT2561/RT61 rev B 802.11g
/0/100/1e/5 eth0 network RTL-8100/8101L/8139 PCI Fast Ethernet Adapter
/0/100/1f bridge 82801EB/ER (ICH5/ICH5R) LPC Interface Bridge
/0/100/1f.1 storage 82801EB/ER (ICH5/ICH5R) IDE Controller
/0/100/1f.2 storage 82801EB (ICH5) SATA Controller
/0/100/1f.3 bus 82801EB/ER (ICH5/ICH5R) SMBus Controller
/0/100/1f.5 multimedia 82801EB/ER (ICH5/ICH5R) AC'97 Audio Controller
/0/2 scsi1 storage
/0/2/0.0.0 /dev/sr0 disk DVD-ROM GDR8164B
/0/2/0.1.0 /dev/cdrom disk DVD writer
/0/3 scsi3 storage
/0/3/0.0.0 /dev/sda disk 750GB ST3750630AS
/0/3/0.0.0/1 /dev/sda1 volume 232GiB EXT4 volume
/0/3/0.0.0/2 /dev/sda2 volume 3814MiB Extended partition
/0/3/0.0.0/2/5 /dev/sda5 volume 3814MiB Linux swap / Solaris partition
```

Con esto concluyo la entrada.