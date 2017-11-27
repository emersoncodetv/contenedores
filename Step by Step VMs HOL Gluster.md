# Step by Step VMs HOL Gluster

<!---
http://www.itzgeek.com/how-tos/linux/centos-how-tos/install-and-configure-glusterfs-on-centos-7-rhel-7.html

http://www.itzgeek.com/how-tos/linux/centos-how-tos/install-and-configure-glusterfs-on-centos-7-rhel-7.html/2
--->

```sh
nano /etc/hosts
```
```txt
192.168.56.103  gluster1.local  gluster1
192.168.56.101  gluster2.local  gluster2
192.168.56.102  client.local client
```

### Instalación para centOS nodos gluster

```sh
yum install -y centos-release-gluster
```

Install GlusterFS:

```sh
yum install -y glusterfs-server
```
```sh
systemctl start glusterd
```
```sh
systemctl status glusterd
```
```sh
systemctl enable glusterd
```

### Configurando el firewall todos nodos y clientes

```sh
# Disable FirewallD
systemctl stop firewalld
systemctl disable firewalld

OR

# Run below command on a node in which you want to accept all traffics comming from the source ip 
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="<ipaddress>" accept'
firewall-cmd --reload
```
## Add Storage:

```sh
fdisk -l
fdisk /dev/sdb
```
![](https://s3-us-west-2.amazonaws.com/public-files-blog/Configure-GlusterFS-on-CentOS-7-Partition-Creation.png)
```sh
mkfs.ext4 /dev/sdb1
mkdir -p /data/gluster
mount /dev/sdb1 /data/gluster
echo "/dev/sdb1 /data/gluster ext4 defaults 0 0" | tee --append /etc/fstab
```

## Configure GlusterFS on CentOS 7:

Before creating a volume, we need to create trusted storage pool by adding gluster2.itzgeek.local. You can run GlusterFS configuration commands on any one server in the cluster will execute the same command on all other servers.

Here I will run all GlusterFS commands in gluster1.itzgeek.local node.

```sh
gluster peer probe gluster2.local
```

```sh
gluster peer status
```

```sh
gluster pool list
```

## Setup GlusterFS Volume:

Create a brick (directory) called “gv0” in the mounted file system on both nodes.

```sh
mkdir -p /data/gluster/gv0
```

Since we are going to use replicated volume, so create the volume named “gv0” with two replicas.

```sh
gluster volume create gv0 replica 2 gluster1.local:/data/gluster/gv0 gluster2.local:/data/gluster/gv0 
## volume create: gv0: success: please start the volume to access data
```

```sh
 gluster volume start gv0
```

```sh 
gluster volume info gv0
```

## Setup GlusterFS Client:

Install glusterfs-client package to support the mounting of GlusterFS filesystems. Run all commands as root user.

```sh
$ su -

### CentOS / RHEL ###
yum install -y glusterfs-client
```

Create a directory to mount the GlusterFS filesystem.

```sh
mkdir -p /mnt/glusterfs
```

```sh
mount -t glusterfs gluster1.local:/gv0 /mnt/glusterfs
```
>If you get any error like below.
```sh
WARNING: getfattr not found, certain checks will be skipped..
Mount failed. Please check the log file for more details.
```
Consider adding Firewall rules for client machine (client.itzgeek.local) to allow connections on the gluster nodes (gluster1.itzgeek.local and gluster2.itzgeek.local). Run the below command on both gluster nodes.
```sh
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="clientip" accept'
```
You can also use gluster2.itzgeek.local instead of gluster1.itzgeek.com in the above command.

Verify the mounted GlusterFS filesystem.

```sh
df -hP /mnt/glusterfs
```

You can also use below command to verify the GlusterFS filesystem.

```sh
cat /proc/mounts
```

Add below entry to /etc/fstab for automatically mounting during system boot.

```sh
nano /etc/fstab
```
>Copiar y pegar el siguiente texto
>>gluster1.local:/gv0 /mnt/glusterfs glusterfs  defaults,_netdev 0 0

## Test GlusterFS Replication and High-Availability:

### GlusterFS Server Side:

To check the replication, mount the created GlusterFS volume on the same storage node.

```sh
[root@gluster1 ~]# mount -t glusterfs gluster2.local:/gv0 /mnt
[root@gluster2 ~]# mount -t glusterfs gluster1.local:/gv0 /mnt
```
Data inside the /mnt directory of both nodes will always be same (replication).

Let’s create some files on the mounted filesystem on the client.local.

```sh
touch /mnt/glusterfs/file1
touch /mnt/glusterfs/file2
```

Verificar los archivos creados en el cliente.

```sh
root@client:~# ls -l /mnt/glusterfs/
```

Test the both GlusterFS nodes whether they have same data inside /mnt.

```sh
[root@gluster1 ~]# ls -l /mnt/
total 0
-rw-r--r--. 1 root root 0 Sep 27  2016 file1
-rw-r--r--. 1 root root 0 Sep 27  2016 file2

[root@gluster2 ~]# ls -l /mnt/
total 0
-rw-r--r--. 1 root root 0 Sep 27 16:53 file1
-rw-r--r--. 1 root root 0 Sep 27 16:53 file2
```
As you know, we have mounted GlusterFS volume from gluster1.local on client.local, now it is the time to test the high-availability of the volume by shutting down the node.

```sh
[root@gluster1 ~]# poweroff
```

Now test the availability of the files, you would see files that we created recently even though the node is down.

```sh
root@client:~# ls -l /mnt/glusterfs/
```

>You may experience slowness in executing commands on the mounted GlusterFS filesystem is due to GlusterFS switchover to gluster2.itzgeek.local when the client.itzgeek.local can not reach gluster1.itzgeek.local.

Create some more files on the GlusterFS filesystem to check the replication.

```sh
touch /mnt/glusterfs/file3
touch /mnt/glusterfs/file4
```

Verify the files count.

```sh
root@client:~# ls -l /mnt/glusterfs/
```

Since the gluster1 is down, all your data’s are now written on gluster2.itzgeek.local due to High-Availability. Now power on the node1 (gluster1.itzgeek.local).

Check the /mnt of the gluster1.itzgeekk.local; you should see all four files in the directory, this confirms the replication is working as expected.

```sh
[root@gluster1 ~]# mount -t glusterfs gluster1.local:/gv0 /mnt

[root@gluster1 ~]# ls -l /mnt/
```




































































