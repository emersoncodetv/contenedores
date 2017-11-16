# Step by Step VM HOL Docker

## 1. Virtualbox Guest additions  

~~~bash
$ su -
uname -r
yum install kernel-uek-devel-`uname -r`
KERN_DIR=/usr/src/kernels/`uname -r`
## Montar la imagen de "Guest additions CD image" desde Virtualbox y click en RUN
reboot
~~~

## 2. Instalar nano, git, nmap y actualizar el equipo

~~~bash
$ su -c 'yum update'
su -
yum install nano
yum install git
yum install nmap -y
nmap
nmap -sV localhost
~~~

## 3. Instalar la última versión de Firefox 
*Descargar Firefox de la pagina oficial*

~~~bash
cd /home/oracle/Downloads
tar -xjvf firefox-XX.X.tar.bz2
cp -r firefox /opt/
ln -sf /opt/firefox/firefox /usr/bin/firefox
~~~

## 4. Instalación de Docker en Oracle Linux 

~~~bash
yum install -y yum-utils 
yum-config-manager --enable ol7_addons
yum-config-manager --enable public_ol7_addons
yum repolist all
yum -y install docker-engine
systemctl enable docker
systemctl start docker
/usr/sbin/usermod -aG docker oracle
docker login container-registry.oracle.com
~~~

## 5. Docker compose installation Oracle Linux 

~~~bash
curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /tmp/docker-compose
cp /tmp/docker-compose /usr/bin/docker-compose
chmod +x /usr/bin/docker-compose
~~~

## 6. SELinux - Tener en cuenta

Security-Enhanced Linux: Un kernel de Linux que integra SELinux impone políticas de control de acceso obligatorias que confinan los programas de usuario y los servidores del sistema, el acceso a archivos y recursos de red.
To put SELinux into enforcing mode:

sestatus | grep -i mode

setenforce 1

Deshabilitelo para evitar confluctos con las reglas del Docker iptables. Para deshabilitar SELinux debe estar en permissive mode: 

~~~bash
setenforce 0
sestatus | grep -i mode
~~~

Deshabilitar el firewalld <br> **(No hacer nunca esto en un ambiente productivo)**

~~~bash
systemctl disable firewalld
systemctl stop firewalld
systemctl status firewalld
~~~

## 7. Instalar Cockpit

~~~bash
$ su -
yum install cockpit 
nano /etc/yum.repos.d/centos.repo
~~~

>#### Copiar y pegar lo siguiente

>>[centos]<br>
>>name=CentOS \$releasever - \$basearch<br>
>>baseurl=http://ftp.heanet.ie/pub/centos/7/os/\$basearch/<br>
>>enabled=1<br>
>>gpgcheck=0<br>

~~~bash
yum repolist
yum update
~~~

[cockpit-docker](http://rpm.pbone.net/index.php3?stat=3&search=cockpit-docker&srodzaj=3&dist[]=94)

~~~bash
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/extras/x86_64/Packages/cockpit-docker-151-1.el7.centos.x86_64.rpm
rpm -ivh cockpit-docker-151-1.el7.centos.x86_64.rpm
~~~

[libssh](https://www.rpmfind.net/linux/rpm2html/search.php?query=libssh)

~~~bash
wget ftp://195.220.108.108/linux/fedora/linux/development/rawhide/Everything/x86_64/os/Packages/l/libssh-0.7.5-3.fc27.x86_64.rpm
yum localinstall libssh-0.7.5-3.fc27.x86_64.rpm
~~~

[cockpit-dashboard](http://rpm.pbone.net/index.php3?stat=3&search=cockpit-dashboard&srodzaj=3&dist[]=94)

~~~bash
wget ftp://mirror.switch.ch/pool/4/mirror/centos/7.4.1708/extras/x86_64/Packages/cockpit-dashboard-151-1.el7.centos.x86_64.rpm
yum localinstall cockpit-dashboard-151-1.el7.centos.x86_64.rpm
~~~

Si el firewalld se esta ejecutando. (Si esta disable omitir este paso)

~~~bash
firewall-cmd --add-port=9090/tcp
firewall-cmd--permanent --add-port=9090/tcp
systemctl enable cockpit.socket
systemctl start cockpit.socket
systemctl start cockpit
systemctl restart cockpit
~~~

Abrir Firefox http://localhost:9090/

## 8. Instalar ngrok 

~~~bash
wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
mv ngrok /opt/
/opt/ngrok http 7777
Ctrl + C
~~~

## 9. Instalar VNC

~~~bash
$ su -
yum install tigervnc-server
su - oracle
$ vncpasswd
$ vncserver :2
$ vncserver -list
su -
cp /lib/systemd/system/vncserver@.service  /etc/systemd/system/vncserver@:2.service
nano /etc/systemd/system/vncserver@\:2.service
~~~

>#### Reemplazar el siguiente texto

>>[Unit]<br>
>>Description=Remote desktop service (VNC) <br>
>>After=syslog.target network.target<br>

>>[Service]<br>
>>Type=forking<br>
>>User=\<USER><br>

>>Clean any existing files in /tmp/.X11-unix environment<br>
>>ExecStartPre=-/usr/bin/vncserver -kill %i<br>
>>ExecStart=/usr/bin/vncserver %i<br>
>>PIDFile=/home/\<USER>/.vnc/%H%i.pid<br>
>>ExecStop=-/usr/bin/vncserver -kill %i<br>

>>[Install]<br>
>>WantedBy=multi-user.target<br>

>#### Por este nuevo <br> Reemplazar \<USER> por el usuario que va a ser usado para el inicio remoto

>>[Unit]<br>
>>Description=Remote desktop service (VNC)<br>
>>After=syslog.target network.target<br>

>>[Service]<br>
>>Type=forking<br>
>>ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'<br>
>>ExecStart=/sbin/runuser -l \<USER> -c "/usr/bin/vncserver %i -geometry 1280x1024"<br>
>>PIDFile=/home/\<USER>/.vnc/%H%i.pid<br>
>>ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'<br>

>>[Install]<br>
>>WantedBy=multi-user.target<br>

~~~bash
systemctl daemon-reload
systemctl start vncserver@:2
systemctl status vncserver@:2
systemctl enable vncserver@:2
reboot
$ ss -tulpn| grep vnc
~~~