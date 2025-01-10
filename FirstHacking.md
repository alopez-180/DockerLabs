# FistHacking - Laboratorio de "dockerlabs.es"

### Primeros pasos:
1. Lo primero que tenemos que hacer es descargar el archivo `.zip` que contiene la máquina vulnerable.

2. Una vez hayamos descargado una máquina en formato `.tar`, veremos que hay un script llamado `auto_deploy.sh` junto con cada máquina, por lo que solamente tendremos que ejecutar ese script para desplegar o borrar el laboratorio.

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

`nmap -sV -sC -p21 -n -vvv -oN services-first 172.17.0.2`

* `-sV`: Este argumento instruye a Nmap para intentar identificar la versión de los servicios que se detectan. Esto puede ser útil para determinar si un servicio en particular tiene vulnerabilidades conocidas.
* `-sC`: Equivalente a --script=default esto lanza algunos scripts simples de reconocimiento.

Con esto obtendremos un poco más de información, como se puede ver en la siguiente imagen: 


Ahora que sabemos el nombre del servicio y la versión, en mi caso, he buscado un poco de información en internet, y he encontrado la siguiente página, donde explica un poco las vulnerabilidades del servicio que esta abriendo el puerto 21.

https://medium.com/@S3Curiosity/understanding-the-vulnerabilities-in-vsftpd-2-3-4-f5e0b8317af5

