# BreackMySSH - Laboratorio de "dockerlabs.es"

Siguiendo con los laboratorios de dockerlabs.es, hoy toca vulnerar la seguridad de la máquina BreakMySSH.

### Primeros pasos:

**1.** Lo primero que tenemos que hacer es descargar el archivo `.tar` que contiene la máquina vulnerable (en este caso la del laboratorio "FirstHacking").

**2.** Una vez hayamos descargado el archivo, veremos que hay un script llamado `auto_deploy.sh` junto con la máquina, por lo que solamente tendremos que ejecutar ese script para desplegar o borrar el laboratorio.

**3.** ❯ `sudo bash auto_deploy.sh breackmyssh.tar`

### Escaneo de Red

Para identificar vulnerabilidades en una máquina, uno de los primeros pasos debería ser realizar un escaneo de puertos. Este proceso, en la mayoría de los casos, puede proporcionarnos pistas clave sobre posibles puntos de acceso.
Por lo tanto, utilizando la herramienta **nmap** vamos a realizar un escaneo de red a la máquina **172.17.0.2**

El comando ha utilizar és el siguiente: 

`nmap -sS -p- -vvv -n -T5 -oN ports-first 172.17.0.2`

* `nmap`: Llamamos a la herramienta nmap.
* `-sS`:  Para descubrir puertos de manera silenciosa y rápida.
* `-p-`:  Los puertos a escanear; el `-` indica que recorremos todo el rango de puertos disponibles, es decir, 65535 puertos.
* `-vvv`: Conforme descubre un puerto, nos lo muestra en pantalla.
* `-n`: No aplicamos la resolución DNS, ya que puede llegar a tardar mucho.
* `-T5`: Usamos la plantilla de escaneo más rápida.
* `-oN`: Guardamos todo en un archivo de formato normal.
* `ports-first`: Priorizamos la obtención de puertos.
* `172.17.0.2`: IP de la máquina vulnerable.

En realidad, se pueden utilizar diversas combinaciones de opciones de nmap. Sin embargo, personalmente prefiero ejecutar este comando con estas opciones inicialmente, y luego, según los resultados, considerar otras combinaciones.

#### Analisis de resultados del primer escaneo

Podemos observar, que con el escaneo de puertos hemos encontrado el puerto **22** abierto ya que esta siendo utilizado por el servicio **ssh**.
Ahora que sabemos que tiene este puerto abierto, vamos ha lanzar otro comando de nmap, para averiguar un poco más de información.

❯ `nmap -sV -sC -p22 -n -vvv -oN services-first 172.17.0.2`

* `-sV`: Instruye a Nmap para intentar identificar la versión de los servicios detectados.
* `-sC`: Equivalente a `--script=default`, lanza algunos scripts simples de reconocimiento.
* `-p22`: Esta vez, especificamos que el puerto, tiene que ser el 22


Después de lanzar este último comando, sabemos que el servicio que está utilizando ssh es **openssh** y además, con la versión 7.7 .
Con esta información, vamos a intentar explotar las vulnerabilidades que pueda tener este servicio.

#### Busqueda de información de vulnerabilidades de OPENSSH 7.7

Antes de comenzar a buscar exploits o utilizar herramientas de Kali, prefiero investigar en internet sobre el servicio que planeamos vulnerar. Este enfoque, en numerosas ocasiones, nos proporciona ideas valiosas sobre cómo proceder.

En mi caso he consultado las siguientes páginas web:

*[Vulnerabilidad en OpenSSH (CVE-2018-15473)](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2018-15473)
*[Openssh-7-7-vulnerability](https://medium.com/@lcolin250/openssh-7-7-vulnerability-b6886e82f6f6)
*[Vulnerabilidad de enumeración de usuarios en OpenSSH](https://www.hackplayers.com/2018/10/enumeracion-de-usuarios-openssh.html)

Resumidamente, he econtrado que OpenSSH hasta la versión 7.7 es propenso a una vulnerabilidad de enumeración de usuarios debido a que no retrasa el rescate de un usuario de autenticación no válido hasta que el paquete que contiene la petición haya sido analizado completamente.

Debido a la diferencia en el tiempo de respuesta para usuarios válidos e inválidos, un atacante puede intentar múltiples combinaciones de nombres de usuario hasta encontrar aquellos que son válidos.

### Explotación de la vulnerabilidad

**1.** El primer paso será buscar si hay algún exploit ya creado para la enumeración de usuarios. Para ello, en la consola de nuestro Kali pondremos: 

❯ `searchsploit OpenSSH 7.7`

Vemos en la imagen, que nos aparecen tres resultados, justamente de lo que queremos "UserName Enumeration"

Como hemos encontrado resultados, vamos a provar si con la herramienta metaexploit tenemos la misma suerte y econtramos uno listo para usar.

*2.** Para abrir la herramienta Metasploit tendremos que escribir: 

❯ `msfconsole`

![Screenshot_4](https://github.com/user-attachments/assets/a73ea429-1824-48cd-b3b4-a8d84e34d3d3)

**3.** Con la consola abierta, procederemos a buscar el exploit para atacar al servicio vsftpd. Para realizar esta búsqueda pondremos:

❯ `search openssh`

Y encontraremos varios disponibles, pero el que nos interesa es la opción **3**, por lo tanto pondremos 

❯ `use 3`

**4.** Ahora que ya tenemos el exploit seleccionado, tendremos que revisar los parámetros obligatorios del exploit. Normalmente, nos pide que pongamos la IP de la máquina vulnerable, el puerto, algún proxy, entre otros. Para revisar lo que nos pide este exploit en concreto, utilizaremos el comando: 

❯ `show options`

En este caso, tenemos que especificar el  `RHOST`, que básicamente es la IP de la máquina a la que queremos acceder. Y también, tenemos que especificar el `USERNAME`, para esta opción, vamos a utilizar un diccionario de usuarios de linux, que por defecto viene instalado en KaliLinux en la ruta **/usr/share/wordlists/metasploit/unix_users.txt**