# Step by Step Arrancando con contenedores

## Descargando imagenes del repositorio publico de docker

>* COMANDOS:
>	* `docker pull <IMAGE NAME> : se contecta al repositorio publico de docker y descarga la imagen que se le pide.`
>	* `docker images : lista todas las imagenes que se han descargado del registro publico o privado.`

~~~sh
$ docker pull fedora
$ docker images

$ docker pull ubuntu
$ docker images

$ docker pull oraclelinux
$ docker images
~~~

## De imagen a contenedor  

Una imagen es un archivo *inerte*, *inmutable*, que es esencialmente una **snapshot de un contenedor**. <br> Ejecucion de un contenedor sencillo.

>* COMANDOS:
>	* `docker run [OPTIONS] IMAGE [COMMAND]: Inicia un nuevo contenedor a partir de una imagen.` <br>
>	* `docker ps: lista todos los conetenedores que estan en ejecución.`
>	* `docker rm <CONTAINER NAME>` o `docker rm <CONTAINER ID>` elimina el contenedor.

>shorthand run 	| Descripción
>------------- 		| -------------
>--interactive , -i	| Keep STDIN open even if not attached
>--tty , -t  			| Allocate a pseudo-TTY
>--rm					| Automatically remove the container when it exits
>
>shorthand ps			| Descripción
>------------- 		| -------------
>--all , -a			| Show all containers (default shows just running)


~~~sh
$ docker images
$ docker run oraclelinux
$ docker ps
~~~
		
~~~sh
$ docker run -i -t oraclelinux bash
exit
$ docker ps -a

$ docker run -i -t oraclelinux bash
exit
$ docker ps -a
~~~
		
~~~sh
$ docker run -i -t --rm oraclelinux bash
exit
$ docker ps -a
~~~

* En el siguiente ejemplo vamos a crear un servidor web improvisado para mostrar como se ejecuta un contenedor de forma mas compleja.

~~~sh
$ su -
mkdir /var/web_data
echo "test" > /var/web_data/test.txt
exit

$ docker run -d -p 9080:9099 --name="my_webserver" \
			-w /opt -v /var/web_data:/opt fedora:latest \
			/usr/bin/python -m SimpleHTTPServer 9099

$ docker run --rm -ti fedora bash
whereis python

$ docker rm my_webserver

$ docker run -d -p 9080:9099 --name="my_webserver" \
			-w /opt -v /var/web_data:/opt fedora:latest \
			/usr/bin/python3.6 -m http.server 9099

$ docker ps
$ netstat -tupln | grep 9080
$ curl localhost:9080

$ su -
echo "demo" > /var/web_data/demo.txt
exit
$ curl localhost:9080
~~~

## Pseudoterminal ver información interna de la imagen

>Se puede usar cualquiera de los dos siempre y cuando el contenedor este 
configurado para usarlos.

>>SHELL = /bin/bash <br>
>>SHELL = /bin/sh <br>
>
>docker exec -i -t \<CONTAINER ID> SHELL # Por ID <br>
>docker exec -i -t \<CONTAINER NAME> SHELL # Por NAME <br>

~~~sh
$ docker ps --format "table {{.ID}}"
$ docker exec -i -t <CONTAINER ID> bash
exit

$ docker ps --format "table {{.Names}}"
$ docker exec -i -t <CONTAINER NAME> bash 
exit

$ docker run --rm -i -t oraclelinux bash
whereis python

$ docker run --rm -i -t oraclelinux whereis python
~~~

## Pseudoterminal a traves del Process ID PID

>* COMANDOS:
>	* `docker inspect <CONTAINER NAME or ID>:  Devuelve información de bajo nivel sobre objetos Docker`

>shorthand inspect		| Descripción
>------------- 		| -------------
>--format , -f				| Format the output using the given Go template

~~~sh
$ docker ps
$ docker inspect --format='{{.State.Pid}}' <CONTAINER NAME>
$ su -
nsenter -m -u -n -i -p -t <Pid>
ls /
exit
~~~

## Revisando un contenedor en funcionamiento

Revisar que contenedores estan corriendo

~~~sh
$ docker ps
~~~

Muestra la configuracion de un contenedor y como fue ejecutado

~~~sh
$ docker inspect <CONTAINER ID>
$ docker inspect <CONTAINER ID> | less
~~~

Si necesitas un dato especifico del contenedor 

~~~sh
$ docker inspect --format='{{.NetworkSettings.IPAddress}}' my_webserver
$ ping <ip>
$ nmap <ip>
~~~

Muestra el puente que esta configurado en el host con docker y el contenedor

~~~sh
$ docker inspect --format='{{.NetworkSettings.Bridge}}' my_webserver
~~~

Muestra los puertos abiertos que tiene el contenedor

~~~sh
$ docker inspect --format='{{.NetworkSettings.Ports}}' my_webserver
~~~

Muestra el mapeo de carpetas con el host

~~~sh
$ docker inspect --format='{{.HostConfig.Binds}}' my_webserver
~~~

El codigo del proceso asociado al contenedor

~~~sh
$ docker inspect --format='{{.State.Pid}}' my_webserver

$ ps -ef | grep <PID>
~~~

## Agregando almacenamiento a un contenedor

### Agregando almacenamiento a un contenedor

- Los contenedores están destinados a ser usados y descartados
- Datos que necesitan estar guardados usualmente están alojados fuera del contenedor
- Agregando storage a un contenedor desde el host:
	* Identificar el punto de montaje con la instrucción VOLUME en el Dockerfile
	* En la linea de comando RUN de docker, usar la opción -v para los puntos de montaje
- Cualquier tipo de almacenaje en Linux puede ser usado (NFS, iSCSI, FC, Gluster...)
	* Atomic hosts dependen de plug-ins para permitir algún tipo de almacenamiento
	* En distribuciones de linux como (Fedora, ubuntu) los tipos de almacenamiento son nativos

### Tecnicas para compartir almacenamiento en Docker

- Compartir datos en contenedores desde el host esta limitado por:
	* Acceso de read-write
	* Compartible entre contenedores o no
- Compartir datos entre contenedores puede usar --volumes-from:
	* `# docker run -v /data:/data:z --name=mydata -it fedora bash` 
		- :z indica que ese volume puede ser compartido con otros contenedores
	* `# docker run --volumes-from=mydata -d fedora touch /data/x`
- Excluyendo un volumen para ser compartido con otros contenedores se hace con :Z:
	* `# docker run -v /vol:/vol:z -v /vol2:/vol2:Z --name=x -it fedora bash`

### Agregando almacenamiento para el Docker Daemon

- Docker Daemon tiene su propio almacenamiento para:
	* Images
	* Contrainers
	* Data cache
- Docker Daemon almacena todo en /var/lib/docker
- Definiendo almacenamiento expansible (como LVM) deja agregar espacio después
	* Cuando se instala linux por primera vez, tienes /var/lib/docker en las particiones de LVM 
	* Crear una nueva partición LVM y apuntar docker daemon alli
	* Apuntar docker daemon a un nuevo directorio usando -g /newpath

~~~sh
# Docker storage

$ docker run it -v /:/host fedora bash
cd /host/tmp
touch abc   # (fails)
exit  
$ docker run -it  -v /:/host --privileged fedora bash
cd /host/tmp
touch abc 	# (succeeds)
exit
~~~

~~~sh
# Docker: almacenamiento entre contenedores

$ ls /vol /vol2
$ docker run -it -v /vol:/vol:z -v /vol2/vol2:z --name=myvols fedora bash
ls /vol /vol2
exit
$ docker run -it --volumes-from=myvols fedora bash
ls /vol /vol2
touch /vol/abc
touch /vol2/abc
ls /vol /vol2
~~~

## Entendiendo linux namespaces

### Entendiendo linux namespaces

- Docker esta basado en Linux Containers (LXC) para aislar:
	* Process table
	* File system
	* Network interfaces
		- El host tiene su propias interfaces de red
		- El contenedor no puede ver otros interfaces de red en el host
	* Inter-process communications (IPC)
	* Cgroups
- Separación entre el proceso del contenedor y el sistema de operaciones

### Abriendo los privilegios del contenedor a el host

* Contenedores privilegiados son unos que son capaces de actuar en el host, asi que puedes administrar, monitorear y solucionar problemas
* Abriendo privilegios individuales del host con `docker run`
	- Process table (`--pid=host`) 
		+ Permite al contenedor ver la tabla de procesos del host
	- File system (`-v /=/host`)
		+ Permite montar un volumen en el punto de montaje en el file system del contenedor
	- Network interfaces (`--net=host`)
		+ Abre todas las interfaces de red del host al contenedor
	- Inter-process communications (`---ipc=host`)
		+ Deja al contenedor ver las IPC facility del host
	- Privileges (`--privileged`)
		+ Abre los privilegios del SELinux y el usuario root en el host

### Accediendo a la tabla de procesos del host

* Use `--pid=host` cuidadosamente
* Permite al contenedor correr comandos para controlar los procesos del host
* Iniciar contenedores con pid:
	- `$ docker run -it --pid=host rhel7/rhel-tools /bin/bash`
* Lista la tabla de procesos:
	- `ps -ef` Muestra todos los procesos corriendo en el host
* Matar un proceso en el host	
	- `killall gnome calculator`

### Accediendo el Host File System

* Montar el host file system en el contenedor (`-v /:/host`)
* Montar un directorio especifico del host al punto de montaje de un contenedor
* Para el acceso total al file system, iniciar el contenedor con lo siguiente:
	- `# docker run -it -v /:/host --privileged fedora bash`
* Agregar, eliminar, o modificar cualquier archivo del host file system:
	- `# touch /host/tmp/newfile`

### Accediendo a las interfaces de red del Host

* De forma predeterminada los contenedores tienen una interface de red separada (eth0 y lo)
* Abrir un acceso directo con la interface de red del host con `--net=host`
* Para el acceso total de las interfaces de red del host, iniciar el contenedor con lo siguiente:
	- `docker run -it --net=host rhel7/rhel-tools bash`
* Comprobar las interfaces de red:
	- `ip a`
* Ejecutar comandos para monitorear las redes

### Accesando al host Inter-Process Communications

* Contenedores tienen separados sus IPC name spaces del host
* Accesando host semaphores, message queues, share memory
* Abriendo el acceso al host IPC environment con lo siguiente:
	- `# docker run -it --ipc=host rhel7/rhel-tools bash`
* Listar IPC message queues, shared memory, y semaphores:
	- `ipcs`

~~~sh
## Host process table demo
$ docker run -it rhel7/rhel-tools /bin/bash
ps -ef
exit
## Terminal 2
$ ping www.google.com

$ docker run -it --pid=host rhel7/rhel-tools /bin/bash
ps -ef | less
ps -ef | grep www.goo
killall -9 ping
exit
~~~

~~~sh
## Host file system demo
$ docker run -it -v /:/host --privileged rhel7/rhel-tools /bin/bash
touch /host/tmp/newfile
ls /home/tmp/newfile
exit
ls /tmpo/newfile
~~~

~~~sh
## Host network interface demo
$ docker run -it rhel7/rhel-tools /bin/bash
ip a
exit
$ docker run -it --net=host rhel7/rhel-tools /bin/bash
ip a | less
exit
~~~

## Ejecutando contenedores con super privilegios

### ¿Por qué usar contenedores con super privilegios?

* Si los contenedores deben estar separados, ¿por qué abrir los privilegios?
	- Permitir herramientas adicionales del host system
	- Herramientas fáciles de eliminar cuando están hechas para volver al estado de ejecución
* ¿Por qué es esto particularmente útil con Atomic hosts?
	- Atomic host esta destinado a ser ligero, entonces no hay muchas herramientas de administración
	- Herramientas del sistema no pueden ser agregados con yum o apt-get en Atomic hosts
* Contenedores con super privilegios con *atomic command* pueden ejecutar:
	- *General tools*
	- Herramientas especificas para el *monitoring* o *logging*

### ¿Cómo son los contenedores con super privilegios cuando se ejecutan (SPCs)

* *Atomic hosts* ofrece el *atomic command* para administrar SPCs
* Con *atomic command*, usted instala, ejecuta o desinstala contenedores
* La opción para `docker run` esta almacenado dentro del SPCs
* Use *atomic info image* para ver el comando predeterminado 
* Disponible SPCs por *RHEL Atomic Host* incluye:
	- rhel-tools: Contiene un un gran conjunto de herramientas para administrar el sistema
	- rsyslog: Incluye *rsyslogd daemon* para el *message logging*
		+ sadc: Tiene una herramienta de recopilacion de datos de las actividades del sistema *(sadc/sar)* 

### Obteniendo el *rhel-tools Container*

* El contenedor *rhel-tools* ofrece cientos de utilidades
* Usados para las soluciones de problemas, mantener y monitorear *Atomic systems*
* Con *atomic*, usted instala, ejecuta, y desinstala *rhel-tools*
* Para obtener el *rhel-tools Container* en un *RHEL Atomic host*, escriba:
	- `# atomic install rhel7/rhel-tools`

### Ejecutando el *rhel-tools Container*

* Para obtener información acerca del *rhel-tools container*, escriba:
	- `# atomic info rhel7/rhel-tools`
	- RUN:

* ~~~sh 
docker run -it --name NAME --privileged \ 
	--ipc=host --net=host --pid=host -e HOST=/host \
	-e NAME=NAME -e IMAGE=IMAGE -v /run:/run \
	-v /var/log:/var/log -v /etc/localtime:/etc/localtime \
	-v /:/host IMAGE
~~~

* Para ejecutar el *rhel-tools Container*, ejecutar:
	- `# atomic run rhel7/rhel-tools`
	- Una vez dentro del contenedor, ejecutar *strace*, *stap*, *tcpdump* u otro commando

### Ejecutando el *rsyslog container*

* para instalar y ejecutar el *rsyslog container*, escriba:
	- `# atomic install rhel7/rsyslog`
	- RUN:

* ~~~sh
docker run --rm --privileged -v /:/host -e HOST=/host \
	-e IMAGE=rhel7/rsyslog -e NAME=rsyslog \
	rhel7/rsyslog /bin/install.sh
	## Instalando el archivo en /host//etc/rsyslog.config 
	## en lugar del directorio vacío existente...
~~~

	- `# atomic run rhel7/rsyslog`
	- `# logger "Verificar el servicio *rsyslogd service*"`
	- `# tail -f /var/log/messages | grep Verificar`

### Obteniendo informacion en el *rsyslog container*

* `# atomic info rhel7/rsyslog`
* RUN: 

* ~~~sh
$ docker run -d --privileged --name NAME --net=host \
	--pid=host -v /etc/pki/rsyslog:/etc/pki/rsyslog \
		-v /etc/rsyslog.conf:/etc/rsyslog.conf \
		-v /etc/sysconfig/rsyslog:/etc/sysconfig/rsyslog \
		-v /etc/rsyslog.d:/etc/rsyslog.d -v /var/log:/var/log \
		-v /var/lib/rsyslog:/var/lib/rsyslog -v /run:/run \
		-v /etc/machine-id:/etc/machine-id \
		-v /etc/localtime:/etc/localtime -e IMAGE=IMAGE \
		-e NAME=NAME --restart=always IMAGE /bin/rsyslog.sh
~~~

* INSTALL:

* ~~~sh
$ docker run --rm --privileged -v /:/host \
	-e HOST=/host -e IMAGE=IMAGE -e NAME=NAME IMAGE \
	/bin/install.sh
~~~

* UNINSTALL:

* ~~~sh
$ docker run --rm --privileged -v /:/host \
	-e HOST=/host -e IMAGE=IMAGE -e NAME=NAME IMAGE
	/bin/uninstall.sh
~~~

### Ejecutando el *sadc container*

Para instalar y ejecutar *sadc container*, escriba:

* `# atomic install rhel7/sadc`
* RUN:

* ~~~sh
$ docker run --rm --privileged --name sadc -v /:/host \
	-e HOST=/host -e IMAGE=rhel7/sadc -e NAME=name rhel7/sadc \
	/usr/local/bin/sysstat-install.sh
	## Instalando archivo en /host//etc/cron.d/sysstat
	## Instalando archivo en /host//etc/sysconfig/sysstat
	## Instalando archivo en /host//etc/sysconfig/sysstat.ioconf
	## Instalando archivo en /host//usr/local/bin/sysstat
~~~

* `# atomic run rhel7/sadc`
* `$ docker exec -it sadc sar`

### Obteniendo información en el *sadc contenedor*

* `# atomic info rhel7/sadc`
* RUN:

* ~~~sh
$ docker run -d --privileged --name NAME \
	-v /etc/sysconfig/sysstat:/etc/sysconfig/sysstat \ 
	-v /etc/sysconfig/sysstat.ioconf:/etc/sysconfig/sysstat.ioconf \ 
	-v /var/log/sa:/var/log/sa -v /:/host -e HOST=/host \
	-e IMAGE=IMAGE -e NAME=NAME --net=host --restart=always \
	IMAGE /usr/local/bin/sysstat.sh
~~~

* INSTALL:

* ~~~sh
$ docker run --rm --privileged --name NAME -v /:/host
	-e HOST=/host -e IMAGE=IMAGE -e NAME=name IMAGE
	/usr/local/bin/sysstat-install.sh
~~~

* UNINSTALL:

* ~~~sh
$ docker run --rm --privileged -v /:/host -e HOST=/host \
	-v /var/log:/var/log -e IMAGE=IMAGE -e NAME=NAME IMAGE \
	/usr/local/bin/sysstat-uninstall.sh
$ docker run -f sadc
~~~

### Hands On Lab HOL

~~~sh
## RHEL Tools DEMO
atomic install rhel7/rhel-tools
atomic run rhel7/rhel-tools
ls / # (Para ver que hay dentro del contenedor)
ps -ef
ip a
cd /host
ls 
~~~

~~~sh
## rsyslog DEMO
atomic install rhel7/rsyslog
atomic run rhel7/rsyslog
ps -ef | grep rsyslog
logger "Verificar el servicio *rsyslogd service*"
tail /var/log/messages | grep Verificar
~~~

~~~sh
## sadc DEMO
atomic run rhel7/sadc
docker exec -it sadc sar
~~~

## Docker logs

~~~sh
$ docker run --rm -v /dev/log:/dev/log oraclelinux:latest \
logger "Sending container massages to the host"

$ su- 
journalctl | grep 'Sending'
journalctl -u docker.service
~~~

## Consultando los contenedores

~~~sh
$ docker run -d -p 9091:9099 --name="my_webserver_play" \
			-w /opt -v /var/web_data:/opt fedora:latest \
			/usr/bin/python3.6 -m http.server 9099

$ curl localhost:9091
$ docker ps
$ docker run hello-world
~~~

Lista el ultimo contenedor creado

~~~sh
$ docker ps -l
~~~

Lista todos los contenedores

~~~sh
$ docker ps -a
~~~

Lista solo los "numeric IDs" de los contenedores

~~~sh
$ docker ps -a -q
$ docker stop my_webserver_play
$ curl localhost:9091
$ docker start my_webserver_play
$ curl localhost:9091
$ docker restart my_webserver_play
~~~

## Instalación de un docker registry en Oracle Linux
Toda la información relacionada con el docker registry la puedes encontrar en: 
[Docker registry](https://docs.docker.com/registry/deploying/)

~~~sh
$ docker run -d -p 5000:5000 --restart=always --name registry registry:latest
$ netstat -tupln | grep 5000
$ docker ps
~~~

Abrir en un browser el siguiente link <br>
[http://localhost:5000/v2/_catalog](http://localhost:5000/v2/_catalog)

## Enviando imágenes a un docker registry local 

~~~sh
$ docker pull oraclelinux:7.4
$ docker tag oraclelinux:7.4 localhost:5000/my-oraclelinux:7.4
$ docker push localhost:5000/my-oraclelinux:7.4
$ docker images
$ docker image remove oraclelinux:7.4
$ docker image remove localhost:5000/my-oraclelinux:7.4
$ docker images
~~~

Abrir en un browser el siguiente link <br>
[http://localhost:5000/v2/_catalog](http://localhost:5000/v2/_catalog)

~~~sh
$ docker pull localhost:5000/my-oraclelinux:7.4
$ docker images
~~~

## Remover contenedores e imagenes

Uso del disco

~~~sh
$ su -
du -sh /var/lib/docker
~~~

Cuanto espacio hay disponible

~~~sh
df -h /var/lib/docker
exit
~~~

no puedes eliminar un conenedor que actualmente es en ejecucion 

~~~sh
$ docker rm my_webserver_play
$ docker stop my_webserver_play
$ docker rm my_webserver_play
~~~

para eliminar varios contenedores 

~~~sh
$ docker rm <CONTAINER ID> <CONTAINER ID> <CONTAINER ID> ...
~~~

para eliminar varias imagenes

~~~sh
$ docker rmi <IMAGE ID> <IMAGE ID> <IMAGE ID> ...
~~~

para listar todos los ID se usa -q ya sea para imagenes o contenedores <br>
Eliminar todos los contenedores que hay en el sistema

~~~sh
$ docker rm -f $(docker ps -a -q)
~~~

Eliminar todas las imagenes que hay en el sistema

~~~sh
$ docker rmi -f $(docker images -a -q)
~~~

## Save container to images 

~~~sh
$ docker run --name="my_whereis" -i -t fedora whereis ssh
$ docker ps
$ docker ps -a
$ docker ps -l -a

$ docker commit -m="whereis ssh on fedora base" \
-a="customName" <CONTAINER ID> fedora_whereisssh

$ docker images | grep fedora_
$ docker run -i -t fedora_whereisssh
$ docker ps -l
$ docker save -o my_fedora_whereisssh.tar fedora_whereisssh
$ ls
~~~

## Transportando imagenes

~~~sh
$ file my_fedora_whereisssh.tar 
~~~

mover el archivo a otro servidor 
Montar el tar en mis imagenes
Vamos a eliminar los contenedores e imagenes para simular un nuevo servidor

~~~sh
$ docker ps
$ docker rm <CONTAINER ID> # fedora_whereisssh
$ docker rmi fedora_whereisssh -f
~~~

Cargemos el tar a nuestro a images

~~~sh
$ docker load -i my_fedora_whereisssh.tar
$ docker images
$ docker run fedora_whereisssh
~~~

## Docker system and healthy

obtener informacion del paquete de docker 

~~~sh
rpm -q docker-engine
~~~

Estado del servicio

~~~sh
systemctl status docker.service
~~~

informacion de docker

~~~sh
docker info
~~~

lista las versiones de cada componente de docker

~~~sh
docker version 
~~~

muestra la informacion de los procesos de docker

~~~sh
docker ps
docker top <CONTAINER ID>
~~~

muestra los cambios en el filesystem despues que el contenedor a iniciado

~~~sh
docker diff <CONTAINER ID>
~~~

muestar la historia de la construccion del contenedor

~~~sh
docker images

docker history <IMAGE ID>
~~~

mostrar el numero de contenedores corriendo en el sistema

~~~sh
docker ps | wc -l
~~~

mostrar el numero de contenedores corriendo y detenidos en el sistema

~~~sh
docker ps -a | wc -l
~~~

ver la informacion de los procesos per se

~~~sh
docker top <CONTAINER ID> -x
~~~

docker events abre un listener para ver que ocurre dentro de docker

terminal 1

~~~
docker events
~~~

terminal 2

~~~sh
docker ps -a
docker rm <CONTAINER ID>
docker rmi hello-world
~~~

Ver lo que ocurrio en la terminal 1

## Construir imagenes a partir de un Dockerfile

Abrir el archivo dockerfileDemo <br>
Primero construimos la imagen mongodb 

~~~sh
$ cd ~
$ mkdir dockerFilesDemo
$ cd dockerFilesDemo
$ mkdir mongodb
$ cd mongodb
$ nano Dockerfile
~~~

>#### Copiar y pegar lo siguiente

>>FROM oraclelinux:7-slim <br>

>>ADD mongodb-org-3.4.repo /etc/yum.repos.d/ <br>
>>RUN yum install -y mongodb-org <br>
>>RUN yum clean all <br>
>>RUN mkdir -p /data/db <br>

>>EXPOSE 27017 <br>

>>CMD ["mongod"] <br>

~~~sh
$ nano mongodb-org-3.4.repo
~~~
>#### Copiar y pegar lo siguiente

>>[mongodb-org-3.4] <br>
>>name=MongoDB Repository <br>
>>baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/ <br>
>>gpgcheck=1 <br>
>>enabled=1 <br>
>>gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc <br>

~~~sh
$ ls
$ docker build --no-cache -t mongo .
$ docker images
$ cd ~
$ mkdir data
$ cd data
$ docker run -d -p 27017:27017 --name mongoRun -v ~/data:/var/lib/mongodb mongo
~~~

Instalando un administrador de mongodb echo en nodejs

~~~sh
$ cd ~
$ cd dockerFilesDemo
$ git clone https://github.com/mrvautin/adminMongo.git && cd adminMongo
$ nano Dockerfile
$ docker build --no-cache -t admin-mongo .
$ docker run -d -e PORT=1234 -p 1234:1234 -it --link mongoRun:mongo --name adminMongo admin-mongo 
$ docker exec -i -t adminMongo /bin/bash
$ docker exec -i -t adminMongo /bin/sh
ping mongo
exit
$ docker exec -i -t mongoRun /bin/bash
mongo
show databases
~~~

>Abrimos el navegador en localhost:1234 <br>
>Agregamos una conexion mongodb://mongo <br>
>Creamos una base de datos wkOracle <br>
>Volvemos a la shell

~~~sh
show databases
~~~

## [ATOMIC IMAGES](https://medium.com/travis-on-docker/microcontainers-tiny-portable-docker-containers-1507e3bf8688)

Contar el numero de packetes que tiene una imagen, pero primero probemos el comando en nuestro servidor host.

~~~sh
$ rpm -qa | wc
$ rpm -qa | less
~~~

Lista los comandos de la shell que se permiten y lista las configuraciones de los paquetes, pero primero probemos el comando en nuestro servidor host.

~~~sh
$ ls /dev /etc
~~~

Muestra la documentacion de man y sus relaciones con otros paquetes, pero primero probemos el comando en nuestro servidor host.

~~~sh
$ ls /usr/share/man/* | less
~~~

Ahora vamos a provar los mismo comandos pero dentro de un contenedor.

~~~sh
$ docker exec -i -t mongoRun /bin/bash
rpm -qa | wc
rpm -qa 
ls /dev /etc
ls /usr/share/man/* | less
type yum
exit
~~~

## Revisar la configuracion predeterminada de red de Docker

~~~sh
$ ip a
$ ifconfig docker0
$ docker run -it oraclelinux bash
ip a
exit
~~~

## Cambiando las configuraciones de red de Docker

~~~sh
$ su -
nano /etc/sysconfig/docker
~~~

>#### Reemplazar el siguiente texto

>>OPTIONS='--selinux-enabled'

>#### Por este nuevo 
>>\#OPTIONS='--selinux-enabled' <br>
>>OPTIONS='--selinux-enabled -b=none'

~~~sh
systemctl restart docker
exit

$ docker run -it oraclelinux bash
ip a 
~~~

> **NOTA:** La interfaz etc0 a desaparecido

~~~sh
exit
$ docker run -it --net=host oraclelinux bash
~~~

> **NOTA:** Puedo activar la red de forma individual en un host

~~~sh
exit
~~~

