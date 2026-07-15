# Ubuntu Server 使用  
## 网络  
默认使用 Network Manager 来管理以太网和 WiFi。
## 以太网  
断开以太网（重启后不再自动连接）：  
```
nmcli dev disconnect iface eth0  
```

打开以太网连接：   
```
nmcli con up id "Ethernet connection 1"
```

关闭以太网连接： 
```
nmcli con down id "Ethernet connection 1"  
```

### 静态 IP  
以太网的连接配置文件为：
```
"/etc/NetworkManager/system-connections/Ethernet connection 1"  
```

其内容为：
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
默认使用 DHCP 来获取动态 IP 地址。
要指定静态 IP 地址，需要更改 "ipv4" 一节成：
```
[ipv4]
   method=manual
   address1=192.168.1.100/24,192.168.1.1
   dns=8.8.8.8;8.8.4.4;
```
address1 行的格式为：  
```
address1=<IP>/<prefix>,<route>
```
### WiFi  
列出可用的 WiFi 存取点：
```
nmcli dev wifi  
```

创建名称为“My cafe"的新连接，使用密码 "caffeine" 连接到 "Cafe Hotspot 1" SSID：  
```
nmcli dev wifi connect "Cafe Hotspot 1" password "caffeine" name "My cafe"
```

列出可用的网络连接：
```
nmcli con list
```

关闭 "My cafe" 网络连接:
```
nmcli con down id "My cafe"
```

打开 "My cafe" 网络连接:
```
nmcli con up id "My cafe"
```

显示 WiFi 打开状态：
```
nmcli nm wifi
```

打开 WiFi：
```
nmcli nm wifi on
```

关闭 WiFi：
```
nmcli nm wifi off
```

## 安装服务器软件包  
服务器软件包按相关性分类成任务。  
### 列出任务  
列出任务清单：  
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
前缀字符表示状态，“i” 表示已安装，“u” 表示未安装。  

### 列出需要安装的软件包
例如，如果你想知道 "lamp-server" (Linux/Apache/MySQL/PHP) 这项任务会安装什么包，可以运行以下命令：  
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

### 安装服务器任务  
以安装 "lamp-server" 任务为例：
```
firefly@firefly:~$ sudo tasksel install lamp-server 
```

运行后，会弹出一个文本对话框，并有进度显示。注意，用 "Ctrl-C" 并不能中断，需要在另一终端用 kill 命令将该进程结束。  
### 密码
#### 系统
用户: root  
密码: firefly  

用户: firefly  
密码: firefly  

### MySQL

用户: root  
密码: firefly  

### 测试

测试 apache 网页服务器：
```
http://<板子IP>/index.html
```

测试 php 是否正常：
```
http://<板子IP>/index.php
```