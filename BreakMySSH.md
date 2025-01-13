# BreakMySSH - Laboratorio de "dockerlabs.es"

Siguiendo con los laboratorios de dockerlabs.es, hoy toca vulnerar la seguridad de la máquina BreakMySSH.

### Primeros pasos:

**1.** Lo primero que tenemos que hacer es descargar el archivo `.tar` que contiene la máquina vulnerable (en este caso la del laboratorio "FirstHacking").

**2.** Una vez hayamos descargado el archivo, veremos que hay un script llamado `auto_deploy.sh` junto con la máquina, por lo que solamente tendremos que ejecutar ese script para desplegar o borrar el laboratorio.

**3.** ❯ `sudo bash auto_deploy.sh breakmyssh.tar`

![image](https://github.com/user-attachments/assets/5892e029-1d13-4e1e-9936-8df324642700)


### Escaneo de Red

Para identificar vulnerabilidades en una máquina, uno de los primeros pasos debería ser realizar un escaneo de puertos. Este proceso, en la mayoría de los casos, puede proporcionarnos pistas clave sobre posibles puntos de acceso. Por lo tanto, utilizando la herramienta **nmap** vamos a realizar un escaneo de red a la máquina **172.17.0.2**.

El comando a utilizar es el siguiente:

`nmap -sS -p- -vvv -n -T5 -oN ports-first 172.17.0.2`

* `nmap`: Llamamos a la herramienta nmap.
* `-sS`: Para descubrir puertos de manera silenciosa y rápida.
* `-p-`: Los puertos a escanear; el `-` indica que recorremos todo el rango de puertos disponibles, es decir, 65535 puertos.
* `-vvv`: Conforme descubre un puerto, nos lo muestra en pantalla.
* `-n`: No aplicamos la resolución DNS, ya que puede llegar a tardar mucho.
* `-T5`: Usamos la plantilla de escaneo más rápida.
* `-oN`: Guardamos todo en un archivo de formato normal.
* `ports-first`: Priorizamos la obtención de puertos.
* `172.17.0.2`: IP de la máquina vulnerable.

![image](https://github.com/user-attachments/assets/79c4dda1-1bb2-4771-8d6b-0993728219cb)

En realidad, se pueden utilizar diversas combinaciones de opciones de nmap. Sin embargo, personalmente prefiero ejecutar este comando con estas opciones inicialmente, y luego, según los resultados, considerar otras combinaciones.

#### Análisis de resultados del primer escaneo

Podemos observar que con el escaneo de puertos hemos encontrado el puerto **22** abierto, ya que está siendo utilizado por el servicio **ssh**. Ahora que sabemos que tiene este puerto abierto, vamos a lanzar otro comando de nmap para averiguar un poco más de información.

❯ `nmap -sV -sC -p22 -n -vvv -oN services-first 172.17.0.2`

* `-sV`: Instruye a Nmap para intentar identificar la versión de los servicios detectados.
* `-sC`: Equivalente a `--script=default`, lanza algunos scripts simples de reconocimiento.
* `-p22`: Esta vez, especificamos que el puerto tiene que ser el 22.

![Screenshot_2](https://github.com/user-attachments/assets/87e34970-fb84-462a-90ec-9a4c755291d1)

Después de lanzar este último comando, sabemos que el servicio que está utilizando ssh es **OpenSSH** y además, con la versión 7.7. Con esta información, vamos a intentar explotar las vulnerabilidades que pueda tener este servicio.

#### Búsqueda de información de vulnerabilidades de OpenSSH 7.7

Antes de comenzar a buscar exploits o utilizar herramientas de Kali, prefiero investigar en internet sobre el servicio que planeamos vulnerar. Este enfoque, en numerosas ocasiones, nos proporciona ideas valiosas sobre cómo proceder.

En mi caso he consultado las siguientes páginas web:

- [Vulnerabilidad en OpenSSH (CVE-2018-15473)](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2018-15473)
- [Openssh-7-7-vulnerability](https://medium.com/@lcolin250/openssh-7-7-vulnerability-b6886e82f6f6)
- [Vulnerabilidad de enumeración de usuarios en OpenSSH](https://www.hackplayers.com/2018/10/enumeracion-de-usuarios-openssh.html)

Resumidamente, he encontrado que OpenSSH hasta la versión 7.7 es propenso a una vulnerabilidad de enumeración de usuarios debido a que no retrasa el rescate de un usuario de autenticación no válido hasta que el paquete que contiene la petición haya sido analizado completamente.

Debido a la diferencia en el tiempo de respuesta para usuarios válidos e inválidos, un atacante puede intentar múltiples combinaciones de nombres de usuario hasta encontrar aquellos que son válidos.

### Explotación de la vulnerabilidad - Listado de Usuarios

**1.** El primer paso será buscar si hay algún exploit ya creado para la enumeración de usuarios. Para ello, en la consola de nuestro Kali pondremos:

❯ `searchsploit OpenSSH 7.7`

![image](https://github.com/user-attachments/assets/d5e12ab4-7864-4428-8500-f2aad12ba200)

Vemos en la imagen que nos aparecen tres resultados, justamente de lo que queremos: "UserName Enumeration".

Como hemos encontrado resultados, vamos a probar si con la herramienta Metasploit tenemos la misma suerte y encontramos uno listo para usar.

**2.** Para abrir la herramienta Metasploit tendremos que escribir:

❯ `msfconsole`

![Screenshot_4](https://github.com/user-attachments/assets/a73ea429-1824-48cd-b3b4-a8d84e34d3d3)

**3.** Con la consola abierta, procederemos a buscar el exploit para atacar al servicio openssh. Para realizar esta búsqueda pondremos:

❯ `search openssh`

Y encontraremos varios disponibles, pero el que nos interesa es la opción **3**, por lo tanto pondremos:

❯ `use 3`

![image](https://github.com/user-attachments/assets/c03cc38a-16d6-483a-abb4-db777bf37c77)

**4.** Ahora que ya tenemos el exploit seleccionado, tendremos que revisar los parámetros obligatorios del exploit. Normalmente, nos pide que pongamos la IP de la máquina vulnerable, el puerto, algún proxy, entre otros. Para revisar lo que nos pide este exploit en concreto, utilizaremos el comando:

❯ `show options`

![image](https://github.com/user-attachments/assets/7d33aa54-b54e-4dfa-b53c-1d8527662a4d)


En este caso, tenemos que especificar el `RHOST`, que básicamente es la IP de la máquina a la que queremos acceder. Y también, tenemos que especificar el `USER_FILE`, para esta opción, vamos a utilizar un diccionario de usuarios de Linux, que por defecto viene instalado en KaliLinux en la ruta **/usr/share/wordlists/metasploit/unix_users.txt**. En la línea de comandos pondremos:

❯ `set RHOST 172.17.0.2`

![image](https://github.com/user-attachments/assets/d4d4055a-0666-4ace-860a-aff6519c5e42)

❯ `set USER_FILE /usr/share/wordlists/metasploit/unix_users.txt`

![image](https://github.com/user-attachments/assets/5236df8b-f029-4241-9e7e-486c9b48e583)


❯ `run`

![image](https://github.com/user-attachments/assets/3f0eb3cb-a01a-4bf1-9a42-576e6842f8a4)


Si lo hemos hecho bien, debería habernos encontrado un listado de usuarios, entre los más importantes tenemos el usuario root del sistema.

### Explotación de la vulnerabilidad - Ataque de Fuerza Bruta

Una vez sabemos algunos de los usuarios que tiene la máquina vulnerable, vamos a intentar entrar usando un ataque de fuerza bruta mediante la herramienta de hydra, que por defecto ya viene instalada en los sistemas operativos KaliLinux. Aparte, lo combinaremos con un archivo que recopila una gran cantidad de contraseñas de las más comunes (conocido como diccionario de contraseñas) llamado rockyou, el cual, también viene por defecto en los sistemas Kali, en la ruta `/usr/share/wordlists/rockyou.txt`.

Para lanzar hydra, solamente necesitaremos poner el siguiente comando:

❯ `sudo hydra -l root -P /usr/share/wordlists/rockyou.txt.gz ssh://172.17.0.2`

Este comando lo que hace es probar todas las contraseñas del diccionario contra el usuario **"root"**. Si cambiamos root por otro usuario hará exactamente lo mismo.

Al lanzar el comando, como podemos ver, nos ha encontrado la contraseña la cual es *estrella*.

![image](https://github.com/user-attachments/assets/5f6b2b1c-f141-4780-86b9-0652bb59b199)

Con esto ya tendriamos acceso a la máquina!!!!

![image](https://github.com/user-attachments/assets/09ab01a5-bb45-47c9-85b1-20ef729c4b7b)



