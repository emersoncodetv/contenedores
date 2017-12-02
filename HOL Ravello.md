# Ravello y Kubernetes

## Worker 3

mkdir /mnt/glusterfs/glusterfs/appsdotwar

cd /mnt/glusterfs/glusterfs/appsdotwar

wget http://www.oracle.com/webfolder/technetwork/tutorials/obe/fmw/wls/12c/03-DeployApps/files/benefits.war

chmod 777 benefits.war 

docker run -d -p 7001:7001 -v /mnt/glusterfs/glusterfs/appsdotwar:/u01/oracle/host emerballen/weblogicapp:app1
 
<!---
docker run -d -p 7002:7002 -v /home/oracle:/u01/oracle/host emerballen/weblogicapp:app2v1
--->

http://k8sworker3-k8sdevaval17nov303-rx8rnvoc.srv.ravcloud.com:7001/benefits/

docker ps

docker stop <CONTAINER ID>

docker rm <CONTAINER ID>

## Master Kubernetes

>#### Copiar y pegar lo siguiente

```sh
$ mkdir kubernetesFiles
$ cd kubernetesFiles/
$ pwd
## /home/centos/kubernetesFiles
```
```sh
$ nano persistentVolume.yaml
```

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
$ kubectl get endpoint

$ kubectl describe deployment appsplusdb 
$ kubectl describe pod >TAB 

$ kubectl delete deployment appsplusdb 
```
