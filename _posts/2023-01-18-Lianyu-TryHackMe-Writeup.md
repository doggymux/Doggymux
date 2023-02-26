---
title: "Lianyu - THMWriteup"
layout: single
excerpt: "Maquina THM fácil, centrada en directory listing en servidores web y estenografía."
show_date: true
classes: wide
header:
  teaser: assets/images/images/teaser/Lianyu/teaser.png
  teaser_home_page: true
  icon: assets/images/icons/HackTheBox-icon.png
categories:
  - writeup
  - Easy
  - TryHackMe
tags:
  - esteganografía
  - magic numbers
---

![lianyu.jpeg](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/lianyu.jpeg)

# Enumeración 🔍

Empezamos listando los servicios que tenemos en los puertos realizamos primero una busqueda rapida para ver que puertos tenemos abiertos en la maquina:

Podemos descubrir los siguientes puertos abiertos:

21,22,80,111

Haciendo una busqueda a los servicios y versiones que tenemos dentro de la maquina podemos encontrar los siguientes datos:

![servicios.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/servicios.png)

# Acceso a la maquina victima 🗝️

Empezamos accediendo a la pagina web que tiene la maquina a través del navegador.

![index.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/index.png)

Nos encontramos con un index basico por lo que vamos a listar los directorios de la web para ver si tenemos alguna otra via. Realizando la busqueda con gobuster encontramos un directorio al cual podemos acceder.

![web1.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/web1.png)

Dentro de esta encontramos lo que parece un usuario llamado vigilante, podemos seguir haciendo directory listing para ver si podemos seguir encontrando nuevas webs accesibles.

![island-web.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/island-web.png)

Como vemos podemos acceder a otra mas llamada 2100.

![web2.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/web2.png)

Dentro de esta veremos un enlace a YouTube roto, si inspeccionamos el código fuente de la maquina encontraremos un mensaje que nos avisara de que podemos aprovechar el ticket de alguna forma, pero la página es estática, es por eso que, podemos probar a seguir listando añadiendo la extensión .ticket.

![web2-inspect.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/web2-inspect.png)

![gobuster3.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/gobuster3.png)

Como vemos conseguimos encontrar un resultado con green_arrow.ticket y una vez dentro conseguimos un codigo el cual podemos intentar tratar para conseguir información.

![web3.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/web3.png)

Podemos usar cyberchef para tratar los datos adquiridos y ver si podemos conseguir información útil.

![cyberchef.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/cyberchef.png)

Teniendo el usuario vigilante y lo que parece una contraseña podemos intentar acceder a los servicios expuestos de la maquina pudiendo acceder al ftp gracias a esto y descargar diferentes archivos.

![ftp vigilante.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/ftp_vigilante.png)

podemos probar a realizar desplazamiento dentro de los directorios del FTP para conseguir mas información gracias a esto conseguimos el nombre de otro usuario.

![ftp movimiento.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/ftp_movimiento.png)

## Analisis de ficheros

Una vez descargados los ficheros podemos ver que no se pueden abrir las imagenes, podemos comprobar con un editor hexadecimal si los ficheros han sido modificados.

Vemos que la imagen Leave_Me_alone.png tiene una cabecera que no corresponde con los archivos png por lo que vamos a modificarla para ver si podemos acceder a esta.

![Untitled](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/Untitled.png)

Una vez modificados podemos abrir el archivo sin problemas.

![Untitled](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/Untitled%201.png)

### Esteganografía

Una vez hemos conseguido esta contraseña podemos probarla en el resto de documentos, como vemos la imagen aa.jpg tampoco es accesible es por ello que puede ser que contenga alguna información que no podemos observar a priori. Con steghide y usando la contraseña que nos aparecia en la imagen podemos ver si conseguimos algun fichero nuevo.

![Untitled](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/Untitled%202.png)

Una vez descomprimimos los archivos podemos ver la siguiente información, dentro del fichero shado se nos da lo que parece una contraseña por lo que podemos probarla con el usuario encontrado anteriormente cuando retrocedimos en los directorios del servidor FTP.

![shado.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/shado.png)

Con esto conseguimos finalmente acceso a la maquina y la flag.

![ssh.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/ssh.png)

# Escalada de privilegios 🚀

Una vez dentro de la maquina lo primero que haremos será comprobar si el usuario tiene la posibilidad de ejecutar algún binario con sudo el cual sea vulnerable.

![sudo -l.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/sudo_-l.png)

Investigando en el man de la app podemos ver que podemos ejecutar comandos como otro usuario, esto combinado con el permiso de sudo nos permite ejecutar una shell como root.

![man pkexec.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/man_pkexec.png)

Aqui vemos el comando en cuestion que tenemos que ejecutar:

![escalada.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/escalada.png)

Una vez hemos accedido como root simplemente vamos a su directorio y tendremos acceso a la flag.

![flag root.png](./../assets/images/images/Machines/2023-01-18-Lianyu-TryHackMe-Writeup/flag_root.png)