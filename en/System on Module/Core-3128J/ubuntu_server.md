# Ubuntu Server Note

## Network
Network Manager is preferred in managing the ethernet and WiFi.

### Ethernet

Disconnect ethernet (No more auto connect after reboot):
```
nmcli dev disconnect iface eth0
```

Turn on ethernet:
```
nmcli con up id "Ethernet connection 1"
```

Turn down ethernet:
```
nmcli con down id "Ethernet connection 1"
```

#### Static IP

The ethernet connection file is
```
"/etc/NetworkManager/system-connections/Ethernet connection 1"
```

with content:
```
[connection]
id=Ethernet connection 1
uuid=d4050376-8790-4b83-ae24-015412398a61
interface-name=eth0
type=ethernet

[ipv6]
method=auto

[ipv4]
method=auto
```

By default, it uses DHCP to fetch IP address.   
To specify static IP address, you can change the "ipv4" section to:
```
[ipv4]
method=manual
address1=192.168.1.100/24,192.168.1.1
dns=8.8.8.8;8.8.4.4;
```

The address1 line has the following format:
```
address1=<IP>/<prefix>,<route>
```

### WiFi

List available WiFi access points:
```
nmcli dev wifi
```

Create a new connection named “My cafe" and then connects it to SSID "Cafe Hotspot 1" using password "caffeine":
```
nmcli dev wifi connect "Cafe Hotspot 1" password "caffeine" name "My cafe"
```

List available connections:
```
nmcli con list
```

Turn down connection named "My cafe":
```
nmcli con down id "My cafe"
```

Turn on connection named "My cafe":
```
nmcli con up id "My cafe"
```

Show wifi on/off status:
```
nmcli nm wifi
```

Switch wifi on:
```
nmcli nm wifi on
```
Switch wifi off:
```
nmcli nm wifi off
```

## Installing Server Packages

Server packages are classified as tasks.

### List Tasks

You can find a list of tasks with command:
```
firefly@firefly:~$ tasksel --list-tasks 
u server    Basic Ubuntu server
i openssh-server    OpenSSH server
u dns-server    DNS server
i lamp-server   LAMP server
u mail-server   Mail server
u postgresql-server PostgreSQL database
u print-server  Print server
u samba-server  Samba file server
u tomcat-server Tomcat Java server
u cloud-image   Ubuntu Cloud Image (instance)
u virt-host Virtual Machine host
u ubuntu-desktop    Ubuntu desktop
u ubuntu-usb    Ubuntu desktop USB
u edubuntu-dvd-live Edubuntu live DVD
u kubuntu-dvd-live  Kubuntu live DVD
u lubuntu-live  Lubuntu live CD
u ubuntu-gnome-live Ubuntu GNOME live CD
u ubuntustudio-dvd-live Ubuntu Studio live DVD
u ubuntu-live   Ubuntu live CD
u ubuntu-usb-live   Ubuntu live USB
u xubuntu-live  Xubuntu live CD
u manual    Manual package selection
```
The prefix char indicates the status with 'i' for installed and 'u' for not installed.

### Show Packages to Be Installed

If you want to know what packages will be intalled by the "lamp-server" task (Linux/Apache/MySQL/PHP server), run:
```
firefly@firefly:~$ tasksel --task-packages lamp-server
apache2-mpm-prefork
mysql-common
php5-json
mysql-client-5.5
libaprutil1-dbd-sqlite3
php5-mysql
mysql-server
ssl-cert
libaprutil1
libapr1
libhtml-template-perl
libdbi-perl
apache2-bin
php5-common
apache2
php5-cli
libdbd-mysql-perl
mysql-server-5.5
libterm-readkey-perl
libaprutil1-ldap
mysql-server-core-5.5
libmysqlclient18
libapache2-mod-php5
libwrap0
apache2-data
tcpd
php5-readline
mysql-client-core-5.5
libaio1
```

### Install Server Task

For example, intall "lamp-server":
```
firefly@firefly:~$ sudo tasksel install lamp-server
```
A text dialog will pop up with progress. Note that you cannot use "Ctrl-C" to stop it. You have to kill it within another console or terminal.

## Password

### System

user: root  
pass: firefly  

user: firefly  
pass: firefly

### MySQL

user: root    
pass: firefly

### Test

Check apache web server:
```
http://<YourBoardIP>/index.html
```

Check if php works:
```
http://<YourBoardIP>/index.php
```