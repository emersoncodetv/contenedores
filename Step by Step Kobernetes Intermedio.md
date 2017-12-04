# HOL Kubernetes Intermedio
## 1. Revisar la sincronización de gluster

### Abrir la Terminal del K8sWorker-3
***

```sh
# Accedemos como root
$ su -

# Nos ubicamos en el siguiente directorio /mnt/glusterfs/glusterfs/appsdotwar
cd /mnt/glusterfs/glusterfs/appsdotwar

# Listamos todos los archivos que hay en el directorio
ls -lrt

# Creamos un archivo binario en el directorio
touch FileAval
```

### Abrir la Terminal del K8sWorker-2
***

```sh
# Accedemos como root
$ su -

# Nos ubicamos en el siguiente directorio /mnt/glusterfs/glusterfs/appsdotwar
cd /mnt/glusterfs/glusterfs/appsdotwar

# Listamos todos los archivos que hay en el directorio
ls -lrt
```

## 2. Probando docker y el contenedor de weblogic

### Abrir la Terminal del K8sWorker-3
***

```sh
docker ps

docker run -d -p 7001:7001 -v /mnt/glusterfs/glusterfs/appsdotwar:/u01/oracle/host emerballen/weblogicapp:app1
```

>##### Punto de control

```sh
docker ps

docker stop <CONTAINER ID>

docker rm <CONTAINER ID>
```

## 3. Jugando con Kubernetes

### Abrir la Terminal del Master de Kubernetes
***

```sh
$ cd kubernetesFiles/

$ ls -lrt
```

```sh
$ nano persistentVolume.yaml
```
>##### Punto de control

```sh
kind: PersistentVolume
apiVersion: v1
metadata:
  name: hostvolume0001
  labels:
    type: local
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/glusterfs/glusterfs/appsdotwar"
```
```sh
$ nano persistentVolumeClaim.yaml
```
```sh
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: glusterclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Creando un deployment en Kubernetes con dos Weblogic y una base de datos
***

```sh
$ nano deployment1.yaml
```

```sh
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: appsplusdb
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webpb
    spec:
      containers:
        - name: cpx
          image: emerballen/weblogicapp:app1
          ports:
            - containerPort: 7001
              hostPort: 7001
          volumeMounts:
            - mountPath: /u01/oracle/host 
              name: appvolume
        - name: dbs
          image: emerballen/weblogicapp:app2v1
          ports:
            - containerPort: 7002
              hostPort: 7002
          volumeMounts:
            - mountPath: /u01/oracle/host 
              name: appvolume
        - name: oracledb
          image: emerballen/oracledemodb
          ports:
            - containerPort: 1521
      volumes:
        - name: appvolume
          persistentVolumeClaim:
            claimName: glusterclaim
```

```sh
$ kubectl create -f deployment1.yaml

$ kubectl get nodes 
$ kubectl get deployments
$ kubectl get pods 

$ kubectl describe deployment appsplusdb 
$ kubectl describe pods <POD NAME>

$ kubectl delete deployment appsplusdb 
```
