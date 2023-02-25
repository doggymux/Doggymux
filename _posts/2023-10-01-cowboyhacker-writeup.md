---
title: "cowboyhacker - THM Writeup"
layout: single
excerpt: "Lame es una maquina en la que se explota el servicio SMB a traves de una vulnerabilidad que permite ejecutar directamente una revershell como root."
show_date: true
classes: wide
header:
  teaser: ""
  teaser_home_page: true
  icon: "assets/images/icons/TryHackMe-icon.png"
categories:
  - Writeup
  - Easy
  - TryHackMe
tags:
  - Hydra
  - SSH
  - FTP
---

![Untitled (Pequeño).png](cowboyhacker/Untitled_(Pequeo).png)

# Enumeración

Empezamos listando los servicios que tenemos en los puertos realizamos primero una busqueda rapida para ver que puertos tenemos abiertos en la maquina:

Podemos descubrir los siguientes puertos abiertos:

21,22,80

Haciendo una busqueda a los servicios y versiones que tenemos dentro de la maquina podemos encontrar los siguientes datos:

![Untitled](cowboyhacker/Untitled.png)

# Acceso a la maquina victima

viendo el servicio FTP en la maquina podemos empezar probando una conexion como usuario anonimo y ver si podemos acceder a algun archivo.

![Untitled](cowboyhacker/Untitled%201.png)

Realizamos la conexion de forma correcta y accedemos a dos archivos los cuales vamos a descargar en nuestra maquina a traves de GET

![Untitled](cowboyhacker/Untitled%202.png)

![Untitled](cowboyhacker/Untitled%203.png)

Como podemos ver hemos adquirido un conjunto de lo que parece contraseñas y el otro documento de task esta firmado por el usuario lin.

Podemos intentar un ataque por fuerza bruta al servicio SSH y comprobar si alguna de las contraseñas adquiridas es correcta.

Podemos utilizar Hydra para realizar la fuerza bruta usando el nombre y el diccionario adquirido.

![Untitled](cowboyhacker/Untitled%204.png)

Una vez sabemos que contraseña es la correcta para realizar el acceso podemos conectarnos y acceder a la flag.

![Untitled](cowboyhacker/Untitled%205.png)

# Escalada de privilegios 🚀

Una vez dentro del usuario podemos empezar a realizar la escalada de privilegios, vamos a empezar comprobando si podemos ejecutar algun binario como sudo y ver si nos permite realizar una escalada de privilegios.

![Untitled](cowboyhacker/Untitled%206.png)

Como vemos tenemos la posibilidad de utilizar tar como sudo podemos comprobar si existe alguna vulnerabilidad que nos permita realizar una escalada de privilegio aprovechando algun exploit.

Usaremos el siguiente comando el cual nos permite realizar una escalada de privilegios:

```bash
tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Una vez ejecutado podemos comprobar que somos root y tenemos acceso a la flag.

![Untitled](cowboyhacker/Untitled%207.png)

![Untitled](cowboyhacker/Untitled%208.png)
