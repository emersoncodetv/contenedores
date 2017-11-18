# Step by Step Arrancando con contenedores

## 1. Descargando imagenes del repositorio publico de docker

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

## 2. De imagen a contenedor  

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

## 3. Pseudoterminal ver información interna de la imagen

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

## 4. Pseudoterminal a traves del Process ID PID

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

## 5. Revisando un contenedor en funcionamiento

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

## 6. Docker logs

~~~sh
$ docker run --rm -v /dev/log:/dev/log oraclelinux:latest \
logger "Sending container massages to the host"

$ su- 
journalctl | grep 'Sending'
journalctl -u docker.service
~~~

## 7. Consultando los contenedores

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

## 8. Instalación de un docker registry en Oracle Linux
Toda la informacion relacionada con el docker registry la puedes encontrar en: 
[Docker registry](https://docs.docker.com/registry/deploying/)

~~~sh
$ docker run -d -p 5000:5000 --restart=always --name registry registry:latest
$ netstat -tupln | grep 5000
$ docker ps
~~~

Abrir en un browser el siguiente link <br>
[http://localhost:5000/v2/_catalog](http://localhost:5000/v2/_catalog)

## 9. Enviando imagenes a un docker registry local 

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

## 10. Remover contenedores e imagenes

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

## 11. Save container to images 

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

## 12. Transportando imagenes

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

## 13. Docker system and healthy

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

## 14. Construir imagenes a partir de un Dockerfile

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

## 15. [ATOMIC IMAGES](https://medium.com/travis-on-docker/microcontainers-tiny-portable-docker-containers-1507e3bf8688)

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

## 16. Revisar la configuracion predeterminada de red de Docker

~~~sh
$ ip a
$ ifconfig docker0
$ docker run -it oraclelinux bash
ip a
exit
~~~

## 17. Cambiando las configuraciones de red de Docker

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

