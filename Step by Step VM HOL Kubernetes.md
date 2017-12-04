# Step by Step VM HOL Kubernetes

<!--
ESTE FUNCIONO
https://severalnines.com/blog/installing-kubernetes-cluster-minions-centos7-manage-pods-services

https://github.com/kubernetes/dashboard

REVISAR
https://www.youtube.com/watch?v=b_fOIELGMDY
-->

## Virtualbox Guest additions  

```sh
$ su -
uname -r
yum install kernel-uek-devel-`uname -r`
KERN_DIR=/usr/src/kernels/`uname -r`
## Montar la imagen de "Guest additions CD image" desde Virtualbox y click en RUN
reboot
```

## Instalar nano, git, nmap y actualizar el equipo

```sh
$ su -c 'yum update'
su -
yum install nano
yum install git
yum install nmap -y
nmap
nmap -sV localhost
```

## Instalar la última versión de Firefox
*Descargar Firefox de la pagina oficial*

```sh
cd /home/oracle/Downloads
tar -xjvf firefox-XX.X.tar.bz2
cp -r firefox /opt/
ln -sf /opt/firefox/firefox /usr/bin/firefox
```

## Prerequisitos

![](https://s3-us-west-2.amazonaws.com/public-files-blog/Kubernetes+Demo.png)

Deshabilite el iptables en cada uno de los nodos par evitar confligtos con Docker iptables rules

```sh
$ systemctl stop firewalld
$ systemctl disable firewalld
```

SELinux - Tener en cuenta

Security-Enhanced Linux: Un kernel de Linux que integra SELinux impone políticas de control de acceso obligatorias que confinan los programas de usuario y los servidores del sistema, el acceso a archivos y recursos de red.
To put SELinux into enforcing mode:

sestatus | grep -i mode

setenforce 1

Deshabilitelo para evitar confluctos con las reglas del Docker iptables. Para deshabilitar SELinux debe estar en permissive mode: 

```sh
$ su -
setenforce 0
sestatus | grep -i mode

nano /etc/sysconfig/selinux
```
>#### Reemplazar el siguiente texto
>>SELINUX=enforcing
>#### Por este nuevo
>>#SELINUX=enforcing <br>
>>SELINUX=permissive

Instalar NTP y este seguro que esta habilitado y ejecutandoce.

```sh
$ yum -y install ntp
$ systemctl start ntpd
$ systemctl enable ntpd
```

**master**

```sh
hostnamectl set-hostname 'K8S-master'
exec bash
```

**minion1**

```sh
hostnamectl set-hostname 'K8S-minion01'
exec bash
```

**minion2**

```sh
hostnamectl set-hostname 'K8S-minion02'
exec bash
```

## Configurando el Kubernetes Master

Los siguientes pasos se deben realizar en el maestro.

.1. Install etcd and Kubernetes through yum:

```sh
$ yum -y install etcd kubernetes
```

.2. Configure etcd to listen to all IP addresses inside /etc/etcd/etcd.conf. Ensure the following lines are uncommented, and assign the following values:

```sh
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"
```

.3. Configure Kubernetes API server inside /etc/kubernetes/apiserver. Ensure the following lines are uncommented, and assign the following values:

```sh
KUBE_API_ADDRESS="--address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBELET_PORT="--kubelet_port=10250"
KUBE_ETCD_SERVERS="--etcd_servers=http://127.0.0.1:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_ADMISSION_CONTROL=""
KUBE_API_ARGS=""
```

.4. Start and enable etcd, kube-apiserver, kube-controller-manager and kube-scheduler:

```sh
$ for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done
```

.5. Define flannel network configuration in etcd. This configuration will be pulled by flannel service on minions:

```sh
$ etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'
```

.6. At this point, we should notice that nodes' status returns nothing because we haven’t started any of them yet:

```sh
$ kubectl get nodes
NAME             LABELS              STATUS
```

## Configurando Kubernetes Minions (Nodes)

Los siguientes pasos se deben realizar en minion1 y minion2 a menos que se especifique lo contrario.

.1. Install flannel and Kubernetes using yum:

```sh
$ yum -y install flannel kubernetes
```

.2. Configure etcd server for flannel service. Update the following line inside /etc/sysconfig/flanneld to connect to the respective master:

```sh
nano /etc/sysconfig/flanneld
```

>#### Reemplazar el siguiente texto
>>FLANNEL_ETCD_ENDPOINTS="http://127.0.0.1:2379"
>#### Por este nuevo
>>\# FLANNEL_ETCD_ENDPOINTS="http://127.0.0.1:2379"<br>
>>FLANNEL_ETCD="http://192.168.56.104:2379"

.3. Configure Kubernetes default config at /etc/kubernetes/config, ensure you update the KUBE_MASTER value to connect to the Kubernetes master API server:

```sh
KUBE_MASTER="--master=http://192.168.56.104:8080"
```

.4. Configure kubelet service inside /etc/kubernetes/kubelet as below: <br>

**minion1**:

```sh
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=192.168.56.105"
KUBELET_API_SERVER="--api_servers=http://192.168.56.104:8080"
# KUBELET_POD_INFRA_CONTAINER="--pod-infra-containerimage=...
KUBELET_ARGS=""
```

**minion2:**

```sh
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=192.168.56.106"
KUBELET_API_SERVER="--api_servers=http://192.168.56.104:8080"
# KUBELET_POD_INFRA_CONTAINER="--pod-infra-containerimage=...
KUBELET_ARGS=""
```

.5. Configurar el apiserver y kubelet.

```
$ su -
nano /etc/kubernetes/apiserver
```

>#### Reemplazar el siguiente texto
>>\# default admission control policies <br>
>>KUBE\_ADMISSION\_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
>#### Por este nuevo
>>\# default admission control policies <br>
>>\# KUBE\_ADMISSION\_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"<br>
>>KUBE\_ADMISSION\_CONTROL=""

.6. Start and enable kube-proxy, kubelet, docker and flanneld services:

```sh
$ for SERVICES in kube-proxy kubelet docker flanneld; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done
```

.7. On each minion, you should notice that you will have two new interfaces added, docker0 and flannel0. You should get different range of IP addresses on flannel0 interface on each minion, similar to below:

**minion1:**

```sh
$ ip a | grep flannel | grep inet
inet 172.17.45.0/16 scope global flannel0
```

**minion2:**

```sh
$ ip a | grep flannel | grep inet
inet 172.17.38.0/16 scope global flannel0
```

.8. Now login to Kubernetes master node and verify the minions’ status:

```sh
$ kubectl get nodes
NAME             LABELS                                  STATUS
192.168.56.105   kubernetes.io/hostname=192.168.56.105   Ready
192.168.56.106   kubernetes.io/hostname=192.168.56.106   Ready
```

You are now set. The Kubernetes cluster is now configured and running. We can start to play around with pods.

### Ejecutar los siguientes comandos en cada minion

```sh
$ nano ~/.kube/config

$ kubectl config set-cluster demo-cluster --server=http://192.168.56.104:8080
$ kubectl config set-context demo-system --cluster=demo-cluster
$ kubectl config use-context demo-system

$ nano ~/.kube/config
$ kubectl get nodes
```

>Si ocurre algun error en la creacion de pods o deployment reiniciar todos los workers

```sh
reboot
```

## Agregando un nuevo nodo

Ejecutamos lo siguiente en uno de los nodos

```sh
$ for SERVICES in kube-proxy kubelet docker flanneld; do
    systemctl stop $SERVICES
    systemctl desable $SERVICES    
    systemctl status $SERVICES
done
```

Copiamos el nodo 

Montamos ese nuevo nodo

Configuramos la red con la nueva ip
192.168.56.107

Establecemos un nuevo nombre al host

```sh
hostnamectl set-hostname 'K8S-minion03'
exec bash
```

.4. Configure kubelet service inside /etc/kubernetes/kubelet as below: <br>

**minion3**:

```sh
nano /etc/kubernetes/kubelet
```

```sh
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=192.168.56.107"
KUBELET_API_SERVER="--api_servers=http://192.168.56.104:8080"
KUBELET_ARGS=""
```


```sh
$ for SERVICES in kube-proxy kubelet docker flanneld; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done
```
