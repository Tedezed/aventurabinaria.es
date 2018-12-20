---
layout: post
title: "Creación, publicación y firma de claves PGP"
date: 2016-05-05 16:25:06 -0700
comments: true
---

En esta entrada vamos a ver los pasos básicos para construir un <a href="https://es.wikipedia.org/wiki/Anillo_de_confianza">anillo de confianza</a> con PGP, certificando la identidad y firmando claves.

## Generar par de claves

Para comenzar generamos un par de claves propio, con nuestro nombre real y correo electronico:
<div class="shellhack"><code>~/.gnupg$ gpg --gen-key</code></div>
&nbsp;
<ol>
	<li>Seleccionamos la opción 4 solo RSA para firmar.</li>
	<li>Tamaño de 4096 bits.</li>
	<li>En mi caso la clave nunca caduca, opción 0.</li>
	<li>Introducimos nuestro nombre real, correo electrónico y un comentario.</li>
</ol>
Una vez introducido las siguientes opciones, nos saldrá el siguiente mensaje.
<div class="shellhack"><code>Es necesario generar muchos bytes aleatorios. Es una buena idea realizar
alguna otra tarea (trabajar en otra ventana/consola, mover el ratón, usar
la red y los discos) durante la generación de números primos. Esto da al
generador de números aleatorios mayor oportunidad de recoger suficiente
entropía.</code></div>
Lo que tendremos que hacer es ejecutar tareas pesadas, como ejecutar una maquina virtual, para generar bytes aleatorios.

Para terminar nos pasara la siguiente información con el ID de nuestra clave:
<div class="shellhack"><code>gpg: clave <strong>E0000000</strong> marcada como de confianza absoluta
claves pública y secreta creadas y firmadas.
gpg: comprobando base de datos de confianza
gpg: 3 dudosa(s) necesaria(s), 1 completa(s) necesaria(s),
modelo de confianza PGP
gpg: nivel: 0 validez: 1 firmada: 0 confianza: 0-, 0q, 0n, 0m, 0f, 1u
pub 40000/E0000000 2015-11-19
Huella de clave = 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000
uid Nuestro_nombre (Comentario) nuestro_correo@gmail.com;</code></div>
También nos informara que la clave como configuramos anteriormente solo sirve para firmar, para cifrar tendremos que crear una subclave:
<div class="shellhack"><code>Tenga en cuenta que esta clave no puede ser usada para cifrar. Puede usar
la orden "--edit-key" para crear una subclave con este propósito.</code></div>


## Agregar una subclave para encriptación

<div class="shellhack"><code>~/.gnupg$ gpg --edit-key nuestro_correo@gmail.com</code></div>
Ejecutamos:
<div class="shellhack"><code>gpg: addkey</code></div>
&nbsp;
<ol>
	<li>Introducimos nuestra contraseña de la clave anterior.</li>
	<li>Seleccionamos la opción 6 RSA solo para cifrar.</li>
	<li>Tamaño de 4096 bits.</li>
	<li>Introducimos la validez.</li>
	<li>De nuevo tendremos que generar bytes aleatorios con la actividad de nuestra maquina.</li>
</ol>
Guardamos y salimos:
<div class="shellhack"><code>gpg: save</code></div>
&nbsp;

<strong>Para ver nuestra clave:</strong>
<div class="shellhack"><code>~/.gnupg$ gpg -k</code></div>


## Exportar e importar clave sin firmar

Exportar clave a archivo:
<div class="shellhack"><code>gpg --output TclavePublica.gpg --export E0000000</code></div>
&nbsp;

Exportar clave a http://pgp.mit.edu/
<div class="shellhack"><code>gpg --send-keys --keyserver pgp.mit.edu E0000000</code></div>
&nbsp;

Importar clave de archivo:
<div class="shellhack"><code>gpg --import clavePublica.gpg</code></div>
&nbsp;

Importar clave de http://pgp.mit.edu/
<div class="shellhack"><code>gpg --keyserver pgp.mit.edu --recv-keys A0000000F</code></div>


## Firma de claves PGP e importación

Después podemos firmar las claves de otras personas para ampliar nuestra zona de confianza, también ellos firmaran la nuestra. En primer lugar hacemos acto de notario y confirmamos la identidad de las personas con su clave y correo.

Una vez echo esto firmamos su clave subida al servidor:
<div class="shellhack"><code>~/.gnupg$ gpg --edit-key A0000000F</code></div>
Firmamos con:
<div class="shellhack"><code>gpg: sign</code></div>
&nbsp;

Introducimos nuestra contraseña para firmar la clave ajena.

Guardamos y salimos:
<div class="shellhack"><code>gpg: save</code></div>
&nbsp;

Enviamos clave firmada al servidor pgp.mit.edu con:
<div class="shellhack"><code>gpg --keyserver pgp.mit.edu --send-keys A0000000F</code></div>

Eliminar clave privada:
<div class="shellhack"><code>gpg --keyserver pgp.mit.edu --delete-secret-key E0000000</code></div>
Eliminar clave publica:
<div class="shellhack"><code>gpg --keyserver pgp.mit.edu --delete-keys E0000000</code></div>
