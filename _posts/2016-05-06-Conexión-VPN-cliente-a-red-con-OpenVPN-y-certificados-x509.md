---
layout: post
title: "Conexión VPN cliente a red con OpenVPN y certificados x509"
date: 2016-05-05 16:25:06 -0700
comments: true
---

En esta entrada explicare como realizar una configuración cliente VPN a red privada (VPN host to net) con un túnel VPN con certificados x509.

En primer lugar instalamos openvpn y openssl para el certificado:
<pre>apt-get install openvpn openssl</pre>
Bajamos los script de ejemplo o lo instalamos desde repositorios:
Opción 1:
<pre>apt-get install easy-rsa</pre>
Opción 2:
<pre>git clone https://github.com/OpenVPN/easy-rsa.git -b 'release/2.x'</pre>
Copiamos los script de ejemplo:
<pre>cd /etc/openvpn/
mkdir easy-rsa
cp -r /root/easy-rsa/easy-rsa/2.0/* easy-rsa</pre>
Dentro de /etc/openvpn/easy-rsa/vars podremos configurar algunas variables de entorno que cargaremos posteriormente, vamos a modificar:
<pre>export KEY_SIZE=2048
export CA_EXPIRE=3650
export KEY_EXPIRE=3650
export KEY_COUNTRY="ES"
export KEY_PROVINCE="Sevilla"
export KEY_CITY="Dos Hermanas"
export KEY_ORG="IESGN"
export KEY_EMAIL="juanmanuel.torres@aventurabinaria.es"
export KEY_OU="Informatica"

export KEY_NAME="vpn_key"
# PKCS11 Smart Card
export PKCS11_MODULE_PATH="/usr/lib/changeme.so"
export PKCS11_PIN=1234
export KEY_CN="ServidorVPN"
</pre>
Cargamos las variables de entorno anteriores:
<pre>root@servidorvpn:/etc/openvpn/easy-rsa# source vars
NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
</pre>
<strong>Creación del certificado:</strong>

Borramos las claves anteriores:
<pre>root@servidorvpn:/etc/openvpn/easy-rsa# ./clean-all</pre>
Creamos dh:
<pre>root@servidorvpn:/etc/openvpn/easy-rsa# ./build-dh
Generating DH parameters, 2048 bit long safe prime, generator 2
.............
root@servidorvpn:/etc/openvpn/easy-rsa# ./build-ca
Generating a 2048 bit RSA private key
.............
Country Name (2 letter code) [ES]:
State or Province Name (full name) [Sevilla]:
Locality Name (eg, city) [Dos Hermanas]:
Organization Name (eg, company) [IESGN]:
Organizational Unit Name (eg, section) [Informatica]:
Common Name (eg, your name or your server's hostname) [ServidorVPN]:
Name [vpn_key]:
Email Address [juanmanuel.torres@aventurabinaria.es]:
</pre>
Creación del certificado para el servidor:
<pre>root@servidorvpn:/etc/openvpn/easy-rsa# ./build-key-server ServidorVPN
Generating a 2048 bit RSA private key
........+++
.............................................+++
writing new private key to 'ServidorVPN.key'
----
Country Name (2 letter code) [ES]:
State or Province Name (full name) [Sevilla]:
Locality Name (eg, city) [Dos Hermanas]:
Organization Name (eg, company) [IESGN]:
Organizational Unit Name (eg, section) [Informatica]:
Common Name (eg, your name or your server's hostname) [ServidorVPN]:
Name [vpn_key]:
Email Address [juanmanuel.torres@aventurabinaria.es]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:<strong>tu_contraseña</strong>
An optional company name []:
</pre>
Creamos el certificado para el cliente:
<pre>root@servidorvpn:/etc/openvpn/easy-rsa# ./build-key cliente-vpn
Generating a 2048 bit RSA private key
...............+++
...................................................+++
writing new private key to 'cliente-vpn.key'
-----
Country Name (2 letter code) [ES]:
State or Province Name (full name) [Sevilla]:
Locality Name (eg, city) [Dos Hermanas]:
Organization Name (eg, company) [IESGN]:
Organizational Unit Name (eg, section) [Informatica]:
Common Name (eg, your name or your server's hostname) [cliente-vpn]:
Name [vpn_key]:
Email Address [juanmanuel.torres@aventurabinaria.es]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
</pre>
Las claves se encuentran en la carpeta: /etc/openvpn/easy-rsa/keys/

<strong>Configuración del servidor VPN:</strong>

Copiamos el paquete de ejemplo de configuración del servidor:
<pre>gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz &gt; /etc/openvpn/server.conf</pre>
Configuración del servidor:
<pre>nano server.conf</pre>
<pre>port 1194
proto udp
dev tun
ca /etc/openvpn/easy-rsa/keys/ca.crt
cert /etc/openvpn/easy-rsa/keys/ServidorVPN.crt
key /etc/openvpn/easy-rsa/keys/ServidorVPN.key
dh /etc/openvpn/easy-rsa/keys/dh2048.pem
server 10.88.88.0 255.255.255.0
push "route 10.99.99.0 255.255.255.0"
ifconfig-pool-persist ipp.txt
keepalive 10 120
comp-lzo
persist-key
persist-tun
status openvpn-status.log
log /var/log/openvpn.log
verb 4
</pre>
Activamos el encaminamiento:
<pre>nano /etc/sysctl.conf</pre>
<pre>net.ipv4.ip_forward=1</pre>
<pre>echo 1 &gt; /proc/sys/net/ipv4/ip_forward</pre>
Reiniciamos el servicio:
<pre>service openvpn restart
service openvpn status</pre>
<strong>Configuración del cliente en el servidor:</strong>
<pre>cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client.ovpn</pre>
Modificamos:
<pre>nano /etc/openvpn/easy-rsa/keys/client.ovpn</pre>
<pre>remote 10.0.0.33 1194</pre>
Tranferimos los certificados al cliente,en mi caso para realizar la prueba lo pasare a otra maquina por scp:
<pre>scp /etc/openvpn/easy-rsa/keys/cliente-vpn.key debian@172.33.23.92:/home/debian/openvpn
scp /etc/openvpn/easy-rsa/keys/cliente-vpn.crt debian@172.33.23.92:/home/debian/openvpn
scp /etc/openvpn/easy-rsa/keys/ca.crt debian@172.33.23.92:/home/debian/openvpn
scp /etc/openvpn/easy-rsa/keys/client.ovpn debian@172.33.23.92:/home/debian/openvpn
</pre>
<strong>Configuración del cliente:</strong>

Movemos los certificados y la configuración:
<pre>sudo mv /home/debian/openvpn/* /etc/openvpn/
sudo mv client.ovpn client.conf
</pre>
Editamos la configuración del cliente:
<pre>nano /etc/openvpn/client.conf</pre>
<pre>client
dev tun
proto udp
remote 10.0.0.33 1194
keepalive 10 120
resolv-retry infinite
nobind
persist-key
persist-tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/cliente-vpn.crt
key /etc/openvpn/cliente-vpn.key
#ns-cert-type server
comp-lzo
verb 4
log /var/log/openvpn.log
</pre>
Quitamos el arranque automatico de OpenVPN:
<pre>update-rc.d -f openvpn remove
</pre>
Reiniciamos el servicio:
<pre>service openvpn restart</pre>
Comprobamos el funcionamiento del tunel:
<pre>root@<strong>servidorvpn</strong>:/etc/openvpn# ifconfig tun0
tun0 Link encap:UNSPEC HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
inet addr:10.99.99.1 P-t-P:10.99.99.2 Mask:255.255.255.255
UP POINTOPOINT RUNNING NOARP MULTICAST MTU:1500 Metric:1
RX packets:8 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:100
RX bytes:672 (672.0 B) TX bytes:0 (0.0 B)

root@<strong>clientevpn</strong>:/etc/openvpn# ifconfig tun0
tun0 Link encap:UNSPEC HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
inet addr:10.99.99.6 P-t-P:10.99.99.5 Mask:255.255.255.255
UP POINTOPOINT RUNNING NOARP MULTICAST MTU:1500 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:100
RX bytes:0 (0.0 B) TX bytes:504 (504.0 B)
</pre>
Añadimos una regla nat al servidor VPN:
<pre>iptables -A POSTROUTING -t nat -s 10.88.88.0/24 -o eth0 -j MASQUERADE
</pre>
Nos conectamos a la maquina de la red interna:
<pre>ssh debian@10.99.99.3</pre>
Con esto ya tendríamos funcionando nuestra conexión host to network con OpenVPN.