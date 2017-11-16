# Kobernetes Basic

<!---
Work
How to deploy a NodeJS app to Kubernetes
https://seanmcgary.com/posts/how-to-deploy-a-nodejs-app-to-kubernetes
-->

<!--+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
++ https://kubernetes.io/docs/user-guide/kubectl/v1.8/#scale
++ https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/
++ https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
++ https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-->

Muestra que servicio de kubernetes se esta ejecutando en cada servidor <br>
Ejecutar el comando en cada nodo

~~~sh
$ ps -e | grep kube
~~~


> **NOTA:** Se puede ejecutar el nodo master y un minion en un unico servididor

Ejecutar comandos en cada nodo <br>
Siempre pide la informacion al master node

~~~sh
## Comando en el master
$ ps -e | grep etcd

$ kubectl --server=192.168.56.104:8080 get service
$ kubectl --server=192.168.56.104:8080 get pod
$ kubectl get service
$ kubectl get pod
~~~


> **NOTAS:** <br>
> CLUSTER-IP es la IP con la cual me puedo conectar a un servicio en particular <br>
> El puerto se muestra en PORT(S) el cual tambien van asociados con el servicio <br>
> Detras de cada uno de esos servicios hay multiples containers que tienen su IP particular

~~~sh
$ kubectl get endpoints
~~~

> **NOTA:** Muestra los endpoint asociados a cada servicio.
> Los internos y los externos es con otro comando

Antes de iniciar debemos validar que los servicios que necesitamos tanto en el master como en los minions se esten ejecutando.

**master**

~~~sh
$ su -
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler docker; do
    systemctl start $SERVICES
    systemctl status $SERVICES 
done
~~~

**minions**

~~~sh
$ su -
for SERVICES in kube-proxy kubelet docker flanneld; do
    systemctl start $SERVICES
    systemctl status $SERVICES 
done
~~~

## MASTER - Creando un deployment

***
<!--$ su -
systemctl status docker
docker run -d --name docker-nginx -p 80:80 nginx
exit
https://seanmcgary.com/posts/how-to-deploy-a-nodejs-app-to-kubernetes
-->

~~~sh
$ mkdir kubernetesFiles
$ cd kubernetesFiles/
$ pwd
## /home/centos/kubernetesFiles

$ nano deployment.yaml
~~~

>#### Copiar y pegar lo siguiente
>```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-server-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-server
    spec:
      containers:
      - name: nginx-server
        image: nginx
        imagePullPolicy: Always
        ports:
        - containerPort: 8000
# vim: set ts=2 expandtab!:
```

~~~sh
$ kubectl create -f deployment.yaml
~~~

### Ejecutar el comando en cada nodo

~~~sh
$ kubectl get deployments
~~~

## MASTER - Creando un service

~~~sh
$ cd /home/centos/kubernetesFiles
$ nano service.yaml
~~~

>#### Copiar y pegar lo siguiente
>```
apiVersion: v1
kind: Service
metadata:
  name: nginx-server
  labels:
    app: nginx-server
spec:
  selector:
    app: nginx-server
  ports:
  - port: 8000
    protocol: TCP
    nodePort: 30061
  type: LoadBalancer
```

~~~sh
$ kubectl create -f service.yaml
~~~

### Ejecutar el comando en cada nodo

~~~sh
$ kubectl get services
~~~

> Abrir un navegador y poner la ip del servidor on el puerto 30061 <br>
> e.g.: 192.168.54.104:30061

~~~sh
$ cd /home/centos/kubernetesFiles
$ mkdir pods
$ cd pods
$ nano mysql.yaml
~~~

>#### Copiar y pegar lo siguiente
>```
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  containers:
    - resources:
        limits :
          cpu: 1
      image: mysql
      name: mysql
      env:
        - name: MYSQL_ROOT_PASSWORD
          # change this
          value: Colombia1!
      ports:
        - containerPort: 3306
          name: mysql
```

~~~sh
$ kubectl create -f mysql.yaml
$ kubectl get pods

## Ejecutar en cada minion
$ su -
docker ps --format "{{.ID}}: {{.Image}}"
exit

$ kubectl describe po mysql
$ cd /home/centos/kubernetesFiles/pods
$ nano mysql-service.yaml
~~~

>#### Copiar y pegar lo siguiente
>```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: mysql
  name: mysql
spec:
  externalIPs:
    - 192.168.56.105
  ports:
    # the port that this service should serve on
    - port: 3306
  # label keys and values that must match in order to receive traffic for this service
  selector:
    name: mysql 
```

~~~sh
$ kubectl create -f mysql-service.yaml

## Ejecutar en cada minion
$ su -
docker ps --format "{{.ID}}: {{.Image}}"
exit

$ kubectl get services
~~~

~~~sh
$ cd /home/centos/kubernetesFiles
$ kubectl get deployments 
$ kubectl scale --replicas=2 -f deployment.yaml

## Ejecutar en cada minion
$ su -
docker ps --format "{{.ID}}: {{.Image}}:" | grep 'mysql\|nginx'
exit

$ kubectl get pod
$ etcdctl ls /registry
$ etcdctl ls /registry/ranges
$ etcdctl get /registry/ranges/servicenodeports
$ etcdctl get /registry/ranges/serviceips
~~~