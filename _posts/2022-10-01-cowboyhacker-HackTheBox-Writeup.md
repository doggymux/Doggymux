---
title: "cowboyhacker - THM Writeup"
layout: posts
excerpt: "Maquina de dificultad baja destinada a aprender a usar Hydra para atacar a un servicio SSH con un diccionario"
show_date: true
classes: wide
header:
  teaser: "![cowboyHacker](https://user-images.githubusercontent.com/63744631/221381505-698061a5-b8ab-43f1-bcc9-ba553f77429e.jpeg)"


  teaser_home_page: true
  icon: "assets/images/icons/TryHackMe-icon.png"
categories:
  - Writeup
  - Easy
  - HackTheBox
tags:
---

![Captura_de_pantalla_2023-01-10_135615](https://user-images.githubusercontent.com/63744631/221379788-1a60632e-1d68-4e75-8d46-191f8e3ed1d6.png)


# Enumeración

Empezamos listando los servicios que tenemos en los puertos realizamos primero una busqueda rapida para ver que puertos tenemos abiertos en la maquina:

Podemos descubrir los siguientes puertos abiertos:

21,22,80

Haciendo una busqueda a los servicios y versiones que tenemos dentro de la maquina podemos encontrar los siguientes datos:

![untitled](https://user-images.githubusercontent.com/63744631/221379902-ed7f8b70-19a2-4d26-9552-a7d5b792b8a5.png)

# Acceso a la maquina victima

viendo el servicio FTP en la maquina podemos empezar probando una conexion como usuario anonimo y ver si podemos acceder a algun archivo.

![Untitled 1](https://user-images.githubusercontent.com/63744631/221379915-c9731374-3607-473a-87e4-68ab87fafe22.png)

Realizamos la conexion de forma correcta y accedemos a dos archivos los cuales vamos a descargar en nuestra maquina a traves de GET

![untitled 2](https://user-images.githubusercontent.com/63744631/221379931-a78bfd0a-0c92-4ce8-bdd8-0006dd5eedac.png)

![Untitled 3](https://user-images.githubusercontent.com/63744631/221379961-fbd55c4f-09be-4813-b5ea-54715d79d99c.png)

Como podemos ver hemos adquirido un conjunto de lo que parece contraseñas y el otro documento de task esta firmado por el usuario lin.

Podemos intentar un ataque por fuerza bruta al servicio SSH y comprobar si alguna de las contraseñas adquiridas es correcta.

Podemos utilizar Hydra para realizar la fuerza bruta usando el nombre y el diccionario adquirido.

<img width="1000" alt="untitled 4" src="https://user-images.githubusercontent.com/63744631/221379979-a1868f65-9c7e-40f5-b780-4b3a8909ab4b.png">

Una vez sabemos que contraseña es la correcta para realizar el acceso podemos conectarnos y acceder a la flag.

<img width="1000" alt="untitled 5" src="https://user-images.githubusercontent.com/63744631/221380005-d84494de-353b-4f2b-b26e-1b43ad25df2f.png">

# Escalada de privilegios

Una vez dentro del usuario podemos empezar a realizar la escalada de privilegios, vamos a empezar comprobando si podemos ejecutar algun binario como sudo y ver si nos permite realizar una escalada de privilegios.

![untitled 6](https://user-images.githubusercontent.com/63744631/221380059-69f2383d-2d4e-4f53-a0e6-ae620b5f9ec9.png)

Como vemos tenemos la posibilidad de utilizar tar como sudo podemos comprobar si existe alguna vulnerabilidad que nos permita realizar una escalada de privilegio aprovechando algun exploit.

Usaremos el siguiente comando el cual nos permite realizar una escalada de privilegios:

```bash
tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Una vez ejecutado podemos comprobar que somos root y tenemos acceso a la flag.
