# FistHacking - Laboratorio de "dockerlabs.es"

### Primeros pasos:
1. Lo primero que tenemos que hacer es descargar el archivo `.tar` que contiene la máquina vulnerable (En este caso la
del laboratorio "FirstHacking").

2. Una vez hayamos descargado el archivo, veremos que hay un script llamado `auto_deploy.sh` junto con la máquina, por lo que solamente tendremos que ejecutar ese script para desplegar o borrar el laboratorio.

3. ❯ `sudo bash auto_deploy.sh firsthacking.tar`

### Escaneo de Red

Una vez tenemos desplegada la máquina vulnerable, vamos a identificar si tiene algún puerto abierto, vulnerable, con el que podamos obtener acceso, para ello vamos a utilizar la herramienta nmap, con el siguiente comando:

`nmap -sS -p- -vvv -n -T5 -oN ports-first 172.17.0.2`

* nmap: Llamamos la herramienta nmap.
* `-sS`:  Para descubrir puertos de manera silenciosa y rápida.
* `-p-`:  Los puertos a escanear en este caso el `-` indica que recorremos todo el rango de puertos disponibles entonces serán 65535 los puertos que se analizaran.
* `-vvv`: Conforme descubre un puerto nos lo muestra por pantalla.
* `-n`: No aplicamos la resolución DNS, ya que puede llegar a tardar mucho.
* `-T5`: Con esto usamos la plantilla de escaneo más rápida.
* `-oN`: Todo lo que obtengamos lo guardaremos en un archivo de formato normal.
* `ports-first`: Priorizamos la obtención de puertos.
* `172.17.0.2`: IP de la máquina vulnerable.

Una vez ejecutemos el comando, nos daremos cuenta de que el puerto 21 está abierto.
Con esta información, vamos a indagar un poco más que servicio está usando el puerto, y si es vulnerable, para ello usaremos un comando muy parecido al anterior: 

❯ `nmap -sV -sC -p21 -n -vvv -oN services-first 172.17.0.2`

* `-sV`: Este argumento instruye a Nmap para intentar identificar la versión de los servicios que se detectan. Esto puede ser útil para determinar si un servicio en particular tiene vulnerabilidades conocidas.
* `-sC`: Equivalente a --script=default esto lanza algunos scripts simples de reconocimiento.

Con esto obtendremos un poco más de información, como se puede ver en la siguiente imagen: 


Ahora que sabemos el nombre del servicio y la versión, en mi caso, he buscado un poco de información en internet, y he encontrado la siguiente página, donde explica un poco las vulnerabilidades del servicio que esta abriendo el puerto 21.

https://medium.com/@S3Curiosity/understanding-the-vulnerabilities-in-vsftpd-2-3-4-f5e0b8317af5

### Explotación de la vulnerabilidad

Ahora que ya sabemos por donde podemos acceder a la máquina, aprovechando las vulnerabilidades encontradas en el escaneo de red, vamos a proceder a acceder y conseguir una reverseshell. 

1- El primer paso, sera buscar si hay algún exploit ya creado. Para ello en la consola de nuestro kali pondremos: 

❯ `searchsploit`

Vemos que nos devuelve un listado con una serie de exploits que podemos usar. 
De la lista, concretamente nos interesan dos, que encajan exactamente con la versión del servició *vsftpd 2.3.4

Una opción es utilizando python y la otra es utilizando la herramienta metasploit. En mi caso personal, me gusta más utilizar metasploit, así que veremos como hacerlo con esta segunda opción. 

2- Para abrir la herramienta metasploit tendremos que escribir: 

❯ `msfconsole`

3- Con la consola abierta, procederemos a buscar el exploit para atacar al servicio vsftpd.
Para realizar esta busqueda pondremos:

❯ `search vsftpd`

Esto nos devolvera dos opciones disponibles, 

* La primer para hacer una denegación de servicios.
* La segunda para poder hacer una backdoor a la maquina.

En este caso usaremos la segunda opción (#1)

4- Para seleccionar el exploit que queramos, utilizaremos el comando:

`use (numero del exploit)` en este caso, es la opción 1, por lo tanto pondremos:

❯ `use 1`

5- Ahora que ya tenemos el exploit seleccionado, tendremos que revisar los parametros obligatorios del exploit, normalmente nos pide que pongamos la ip de la maquina vulnerable, el puerto, algún proxi, entre otras. 

Para revisar lo que nos pide este exploit en concreto, utilizaremos el comando: 

❯ `show options`






En este caso, solo nos pide que pongamos el *RHOST, que basicamente es la ip de la maquina a la que queremos acceder. 

Por lo tanto vamos a poner: 

❯ `set RHOST 172.17.0.2`


Y con esto tan sencillo, ya podemos lanzar el exploit con el comado:

❯ `run`

Poco a poco nos iran apereciendo mensajes, con el processo que esta llevando acabo el exploit. 
Y en tan solo en unos segundos, veremos que tendremos acceso a la maquina, estando en una reverseshell

