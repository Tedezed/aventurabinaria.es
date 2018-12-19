---
layout: post
title: "Asociar VIP (Virtual IP) a IP flotante OpenStack"
date: 2016-05-22 16:25:06 -0700
comments: true
---

En esta entrada tratare un problemas al que muchos nos hemos enfrentado al montar un cluster Activo/Activo o Activo/Pasivo con VIP dentro de de OpenStack. El problema reside en que la VIP que asignas es normalmente de una subnet de neutron, para poder acceder a este recurso podremos acceder desde otra maquina virtual en la misma subnet o asociando la IP de la VIP a una IP flotante.

<a href="http://www.aventurabinaria.es/wp-content/uploads/2016/05/VIP-OPENSTACK.png"><img src="http://www.aventurabinaria.es/wp-content/uploads/2016/05/VIP-OPENSTACK.png" alt="VIP-OPENSTACK" width="581" height="230" class="alignnone size-full wp-image-867" /></a>

Para poder realizar la asociación correctamente necesitaremos; la VIP, id IP flotante, id port de la VIP.

En primer lugar listamos las redes con neutron, para consultar la id de la red

	(cli-openstack)debian-user@debian:~/$ neutron net-list
	+--------------------------------------+--------------------------+--------------------------------------------------+
	| id                                   | name                     | subnets                                          |
	+--------------------------------------+--------------------------+--------------------------------------------------+
	| 9734556d-XXXX-XXXX-XXXX-fdad435fff18 | red de usuario.xxxx | 56693ca1-XXXX-XXXX-XXXX-ee8074bb13b1 10.0.0.0/24 |
	| a86fc437-XXXX-XXXX-XXXX-f37a7b05537f | ext-net                  | 3c8cdfb0-XXXX-XXXX-XXXX-c6c3e228b81c             |
	+--------------------------------------+--------------------------+--------------------------------------------------+

Creamos un puerto para la VIP **`neutron port-create {net-id} --fixed-ip ip_address={ip}`**

	neutron port-create 9734556d-XXXX-XXXX-XXXX-fdad435fff18 --fixed-ip ip_address=10.0.0.38

Podremos ver el resultado con

	(cli-openstack)debian-user@debian:~/Descargas$ neutron port-list | grep 10.0.0.38
	2821603c-POPP-POPP-POPP-bd2349b29e63 | | fa:16:3e:94:a9:bd | {"subnet_id":"56693ca1-XXXX-XXXX-XXXX-ee8074bb13b1","ip_address":"10.0.0.38"}

Para poder asociar la IP flotante necesitaremos su id, para ello podemos listar las existentes con

	(cli-openstack)debian-user@debian:~/$ neutron floatingip-list
	+--------------------------------------+------------------+---------------------+--------------------------------------+
	| id                                   | fixed_ip_address | floating_ip_address | port_id                              |
	+--------------------------------------+------------------+---------------------+--------------------------------------+
	| 03eeacd9-XXXX-XXXX-XXXX-71a1182ee838 |                  | 172.22.205.249      |                                      |
	| 0f36bd32-XXXX-XXXX-XXXX-e41867c5b8d4 | 10.0.0.50        | 172.22.205.246      | 0a10a744-XXXX-XXXX-XXXX-b8ecdcd8bac0 |
	| 1f8d124a-XXXX-XXXX-XXXX-101c148a348d | 10.0.0.44        | 172.22.205.241      | 8c306c2b-ZZZZ-ZZZZ-ZZZZ-b64bb5753e6f |
	| 2cc63597-XXXX-XXXX-XXXX-45e8e85034fa | 10.0.0.46        | 172.22.205.243      | 2f56c5a4-YYYY-YYYY-YYYY-541916af4b92 |
	| 61b53f55-XXXX-XXXX-XXXX-4ca9c90479d5 | 10.0.0.45        | 172.22.205.242      | 8b628bdc-VVVV-VVVV-VVVV-b1608f29624d |
	| 75619867-CCCC-CCCC-CCCC-2ac0907329ee | 10.0.0.48        | 172.22.205.244      | 469b1a30-XXXX-XXXX-XXXX-ba487da79e3f |
	| acd9ba03-RRRR-RRRR-RRRR-f535dcf0c769 | 10.0.0.43        | 172.22.205.240      | 40f6ad24-JJJJ-JJJJ-JJJJ-7be049772175 |
	| d6f96986-EEEE-EEEE-EEEE-563823df3216 |                  | 172.22.205.248      |                                      |
	| e4d55da9-WWWW-WWWW-WWWW-308d9b147b13 | 10.0.0.53        | 172.22.205.247      | 8da8d613-BBBB-BBBB-BBBB-96e89e31d8f5 |
	| ea9033b1-XXXX-XXXX-XXXX-6dc56a6b9cea | 10.0.0.52        | 172.22.205.245      | c219e492-NNNN-NNNN-NNNN-62ee771c5700 |
	+--------------------------------------+------------------+---------------------+--------------------------------------+

Finalmente la asociamos con la IP flotante con la VIP con **`neutron floatingip-associate --fixed-ip-address {ip} {float-ip-uuid} {port-uuid}`**

	neutron floatingip-associate --fixed-ip-address 10.0.0.38 d6f96986-EEEE-EEEE-EEEE-563823df3216 2821603c-POPP-POPP-POPP-bd2349b29e63

Resultado final

	+--------------------------------------+------------------+---------------------+--------------------------------------+
	| id                                   | fixed_ip_address | floating_ip_address | port_id                              |
	+--------------------------------------+------------------+---------------------+--------------------------------------+
	| 03eeacd9-XXXX-XXXX-XXXX-71a1182ee838 |                  | 172.22.205.249      |                                      |
	| 0f36bd32-XXXX-XXXX-XXXX-e41867c5b8d4 | 10.0.0.50        | 172.22.205.246      | 0a10a744-XXXX-XXXX-XXXX-b8ecdcd8bac0 |
	| 1f8d124a-XXXX-XXXX-XXXX-101c148a348d | 10.0.0.44        | 172.22.205.241      | 8c306c2b-ZZZZ-ZZZZ-ZZZZ-b64bb5753e6f |
	| 2cc63597-XXXX-XXXX-XXXX-45e8e85034fa | 10.0.0.46        | 172.22.205.243      | 2f56c5a4-YYYY-YYYY-YYYY-541916af4b92 |
	| 61b53f55-XXXX-XXXX-XXXX-4ca9c90479d5 | 10.0.0.45        | 172.22.205.242      | 8b628bdc-VVVV-VVVV-VVVV-b1608f29624d |
	| 75619867-CCCC-CCCC-CCCC-2ac0907329ee | 10.0.0.48        | 172.22.205.244      | 469b1a30-XXXX-XXXX-XXXX-ba487da79e3f |
	| acd9ba03-RRRR-RRRR-RRRR-f535dcf0c769 | 10.0.0.43        | 172.22.205.240      | 40f6ad24-JJJJ-JJJJ-JJJJ-7be049772175 |
	| d6f96986-EEEE-EEEE-EEEE-563823df3216 | 10.0.0.38        | 172.22.205.248      | 2821603c-POPP-POPP-POPP-bd2349b29e63 |
	| e4d55da9-WWWW-WWWW-WWWW-308d9b147b13 | 10.0.0.53        | 172.22.205.247      | 8da8d613-BBBB-BBBB-BBBB-96e89e31d8f5 |
	| ea9033b1-XXXX-XXXX-XXXX-6dc56a6b9cea | 10.0.0.52        | 172.22.205.245      | c219e492-NNNN-NNNN-NNNN-62ee771c5700 |
	+--------------------------------------+------------------+---------------------+--------------------------------------+ 

