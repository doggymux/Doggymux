---
title: "Lame - HTB Writeup"
layout: single
excerpt: "Lame es una maquina en la que se explota el servicio SMB a traves de una vulnerabilidad que permite ejecutar directamente una revershell como root."
show_date: true
classes: wide
header:
  teaser: "https://user-images.githubusercontent.com/63744631/221379077-8faa643c-3729-4623-b3bf-4a13a93a90bd.png"

  teaser_home_page: true
  icon: "assets/images/icons/HackTheBox-icon.png"
categories:
  - Writeup
  - Easy
  - HachTheBox
tags:
  - Samba
---


![Lame 1](https://user-images.githubusercontent.com/63744631/221377504-04449bd0-99a6-4342-b18d-36536fce7f45.png)


# Reconocimiento

Empezamos realizando un primer reconocimiento a la maquina para detectar que puertos tenemos abiertos.

```json
[*] Extracting information...

    [*] IP Address: 10.129.211.36
    [*] Open ports: 21,22,139,445,3632

[*] Ports copied to clipboard
```

Realizando un escaner mas exhaustivo podemos alcanzar a ver que tenemos acceso anonimo al servidor FTP de la maquina.

```python
❯ sudo nmap -sCV -T5 --min-rate 5000 -p 21,22,139,445,3632 10.129.211.36 -oN openPorts -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-25 18:12 CET
Nmap scan report for 10.129.211.36
Host is up (0.046s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.67
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 600fcfe1c05f6a74d69024fac4d56ccd (DSA)
|_  2048 5656240f211ddea72bae61b1243de8f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2023-02-25T12:13:21-05:00
|_clock-skew: mean: 2h30m24s, deviation: 3h32m09s, median: 23s
|_smb2-time: Protocol negotiation failed (SMB2)
```

Haciendo una busqueda en searchsploit podemos encontrar que esta versión de vsftpd tiene un Backdoor Command Execution lo cual podremos probar mas adelante aunque es uno de los accesos mas posibles a la maquina.

<img width="568" alt="Untitled" src="https://user-images.githubusercontent.com/63744631/221377532-c9ffa2bc-c8aa-48f8-ae3f-e674d9acf8bf.png">


# Analisis SMB

Si hacemos una pequeña busqueda en smb para ver si tenemos alguna posible conexión encontramos una carpeta con lectura y escritura.

<img width="638" alt="Untitled 1" src="https://user-images.githubusercontent.com/63744631/221377541-0a3d8834-54e8-4703-8f34-6eb4629dfe49.png">

Podemos conectarnos directamente y ver que tenemos algunos archivos dentro de esta carpeta aunque ninguno potencialmente interesante.

<img width="395" alt="Untitled 2" src="https://user-images.githubusercontent.com/63744631/221377546-34ea270b-4e04-4832-9e2b-6bdf13f3b8c7.png">

Tras conectarnos a ambos servicios comprobamos si existe algun exploit para esta versión de smb podemos ver otro exploit que nos permite realizar una ejecución remota de comandos.

<img width="649" alt="Untitled 3" src="https://user-images.githubusercontent.com/63744631/221377552-37da6422-381a-495b-9053-ca67907bb554.png">

Searchsploit nos permite copiarnos el codigo al directorio de trabajo ademas de mostrarnos el CVE que se explota.

<img width="480" alt="Untitled 4" src="https://user-images.githubusercontent.com/63744631/221377557-c6530a2b-b483-48fe-a385-605f10ca290d.png">

Realizando una busqueda rapida en google podemos llegar a un repositorio de github el cual tiene un exploit escrito en python para realizar una reverse shell al equipo en cuestión. 

[https://github.com/amriunix/CVE-2007-2447](https://github.com/amriunix/CVE-2007-2447)

## Explotación

<img width="341" alt="Untitled 5" src="https://user-images.githubusercontent.com/63744631/221377562-864eaaca-3fb0-487c-8b7c-24369f218af8.png">

<img width="322" alt="Untitled 6" src="https://user-images.githubusercontent.com/63744631/221377566-524b51b8-f3fa-453f-98c5-267147eeb87c.png">

Una vez dentro ya podremos localizar las flags las cuales se encuentran en el directorio del usuario y en el directorio de Root.

<img width="324" alt="Untitled 7" src="https://user-images.githubusercontent.com/63744631/221377572-aff4be5b-79a1-4106-ae06-8024676da515.png">
