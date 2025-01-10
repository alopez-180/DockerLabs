# FistHacking - Laboratorio de "dockerlabs.es"

### Primeros pasos:
**1.** Lo primero que tenemos que hacer es descargar el archivo `.tar` que contiene la máquina vulnerable (en este caso la del laboratorio "FirstHacking").

**2.** Una vez hayamos descargado el archivo, veremos que hay un script llamado `auto_deploy.sh` junto con la máquina, por lo que solamente tendremos que ejecutar ese script para desplegar o borrar el laboratorio.

**3.** ❯ `sudo bash auto_deploy.sh firsthacking.tar`

![Screenshot_1](https://github.com/user-attachments/assets/795a3472-d151-4dfe-83df-275897639b98)

### Escaneo de Red

Una vez tenemos desplegada la máquina vulnerable, vamos a identificar si tiene algún puerto abierto, vulnerable, con el que podamos obtener acceso. Para ello, vamos a utilizar la herramienta nmap, con el siguiente comando:

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

![Screenshot_2](https://github.com/user-attachments/assets/828ea126-9af2-4f2a-8dae-ec0de382687b)

Una vez ejecutemos el comando, nos daremos cuenta de que el puerto 21 está abierto. Con esta información, vamos a indagar un poco más sobre qué servicio está usando el puerto y si es vulnerable. Para ello, usaremos un comando muy parecido al anterior:

❯ `nmap -sV -sC -p21 -n -vvv -oN services-first 172.17.0.2`

* `-sV`: Instruye a Nmap para intentar identificar la versión de los servicios detectados.
* `-sC`: Equivalente a `--script=default`, lanza algunos scripts simples de reconocimiento.

Con esto obtendremos más información, como se puede ver en la siguiente imagen: 

![image](https://github.com/user-attachments/assets/26279526-74a6-4235-9547-1519992f7703)

Ahora que sabemos el nombre del servicio y la versión, busqué un poco de información en internet y encontré la siguiente página, donde explican las vulnerabilidades del servicio que abre el puerto 21.

[Understanding the Vulnerabilities in vsftpd 2.3.4](https://medium.com/@S3Curiosity/understanding-the-vulnerabilities-in-vsftpd-2-3-4-f5e0b8317af5)

### Explotación de la vulnerabilidad

Ahora que sabemos por dónde podemos acceder a la máquina, aprovechando las vulnerabilidades encontradas en el escaneo de red, vamos a proceder a conseguir una reverse shell. 

**1.** El primer paso será buscar si hay algún exploit ya creado. Para ello, en la consola de nuestro Kali pondremos: 

❯ `searchsploit`

![image](https://github.com/user-attachments/assets/02a9dd64-a432-46a7-8daf-649b4bc1ba01)


Vemos que nos devuelve un listado con una serie de exploits que podemos usar. De la lista, concretamente nos interesan dos que encajan exactamente con la versión del servicio *vsftpd 2.3.4*.

Una opción es utilizando Python y la otra es utilizando la herramienta Metasploit. En mi caso personal, prefiero utilizar Metasploit, así que veremos cómo hacerlo con esta segunda opción. 

**2.** Para abrir la herramienta Metasploit tendremos que escribir: 

❯ `msfconsole`

![Screenshot_4](https://github.com/user-attachments/assets/a73ea429-1824-48cd-b3b4-a8d84e34d3d3)


**3.** Con la consola abierta, procederemos a buscar el exploit para atacar al servicio vsftpd. Para realizar esta búsqueda pondremos:

❯ `search vsftpd`

Esto nos devolverá dos opciones disponibles: 

* La primera para hacer una denegación de servicio.
* La segunda para poder hacer una backdoor a la máquina.

En este caso, usaremos la segunda opción (#1).

![Screenshot_5](https://github.com/user-attachments/assets/dc50c913-f5fb-4ea4-b5a2-d760524341e7)


**4.** Para seleccionar el exploit que queramos, utilizaremos el comando:

`use (número del exploit)` en este caso, es la opción 1, por lo tanto pondremos:

❯ `use 1`

**5.** Ahora que ya tenemos el exploit seleccionado, tendremos que revisar los parámetros obligatorios del exploit. Normalmente, nos pide que pongamos la IP de la máquina vulnerable, el puerto, algún proxy, entre otros. Para revisar lo que nos pide este exploit en concreto, utilizaremos el comando: 

❯ `show options`

![Screenshot_6](https://github.com/user-attachments/assets/2bedba15-76c2-46e3-8c7e-d7810321a8ef)


En este caso, solo nos pide que pongamos el `RHOST`, que básicamente es la IP de la máquina a la que queremos acceder. Por lo tanto, pondremos: 

❯ `set RHOST 172.17.0.2`

![Screenshot_7](https://github.com/user-attachments/assets/23209a39-a19c-4732-8275-88345e61dffb)


Y con esto tan sencillo, ya podemos lanzar el exploit con el comando:

❯ `run`

![Screenshot_8](https://github.com/user-attachments/assets/c034f2da-74df-44c6-a0e8-c08a74fd6844)


Poco a poco nos irán apareciendo mensajes con el proceso que está llevando a cabo el exploit. En tan solo unos segundos, veremos que tendremos acceso a la máquina, estando en una reverse shell.

![Screenshot_11](https://github.com/user-attachments/assets/c50a22d9-62c7-4698-b895-3447dcadb72b)
![Screenshot_10](https://github.com/user-attachments/assets/2a4aec07-0a29-4ceb-9014-b02ff85c4d17)
![Screenshot_9](https://github.com/user-attachments/assets/e341d9ba-a5d6-4a96-958f-73c3940ee0aa)


