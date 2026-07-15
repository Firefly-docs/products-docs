# 创建 Ubuntu 根文件系统  

## 使用 miniroot 来创建并引导系统  

miniroot 的主页是：http://androtab.info/miniroot/  
miniroot 是个非常小巧的 shell 环境，用来安装和引导其它根文件系统，例如 Ubuntu, Gentoo, Arch Linux 等，这些系统可以在内核支持的存储设备的根或子目录上。这意味着我们能够从开发板的 eMMC Flash, 外置 TF 卡或 U 盘上安装多个系统，而且方便地切换系统，而不用修改并烧写 parameter 文件。  
miniroot 需要使用串口线来调试，参见[《串口调试》](debug.md)一文。另外在下载系统映像时需要使用以太网，当然，也可以预先下载到移动存储设备上。  

### 准备  

请先备份好开发板及相关存储设备上的数据，以免操作失误或其它不可预见的因素带来的数据丢失。  
首先确保开发板已经烧写了可以正常工作的固件，然后下载以下映像文件：

* [misc.img](https://pan.baidu.com/s/14HAKpTKrm8ahPF-WVoanUQ) (提取码:3128)
* [linux-boot-miniroot.img](https://pan.baidu.com/s/18LtIZMUghhFPiQeGaI98nQ) (提取码:3128)

如果开发板安装的是 Android 或双系统固件，则将 linux-boot-miniroot.img 写到 recovery 分区，misc.img 写到 misc 分区。  
如果开发板安装的是 Linux固件，则将 linux-boot-miniroot.img 写到 boot 分区。  
miniroot 初次启动后，会进入 shell，在串口终端上可以见到提示符：  

```
miniroot#
```

然后开始配置网络，如果是 DHCP 网络：  

```
miniroot# udhcpc  
```

否则就要手工配置网络参数（将192.168.1.* 替换成实际使用的网络地址）：  

```
miniroot# ip addr add 192.168.1.2/24 broadcast + dev eth0  
miniroot# ip link set dev eth0 up  
miniroot# ip route add default via 192.168.1.1
miniroot# echo nameserver 192.168.1.1 > /etc/resolv.conf  
```

miniroot 支持从目录里启动，这就意味着根文件系统的放置位置很灵活，而且可以方便地支持多种 Linux 发行版启动。  
注意，由于调试串口与 TF 卡接口有信号引脚共用，因此不能同时使用。 下面用 U 盘第一分区作为系统存储，创建 ext4 文件系统并挂载到 /mnt，ubuntu 将解压到 /mnt/ubuntu 下：

```
miniroot# mkfs.ext4 -E nodiscard /dev/sda1
miniroot# mount /dev/sda1 /mnt
```

一般需要保证此分区有 4G 以上的剩余空间。  

### 下载和解压 ubuntu-core  

ubuntu-core 是最小的根文件系统，在安装之后根据需要再设置桌面或服务器环境。  
下载并解压到 /mnt :  

```
miniroot# cd /mnt
miniroot# wget -P /mnt http://cdimage.ubuntu.com/ubuntu-core/releases/15.04/release/ubuntu-core-15.04-core-armhf.tar.gz
miniroot# mkdir /mnt/ubuntu
miniroot# tar -xpzf /mnt/ubuntu-core-15.04-core-armhf.tar.gz -C /mnt/ubuntu
```

### 启动 Ubuntu  

* 设置主机名称  

```
miniroot# echo ubuntu > /mnt/ubuntu/etc/hostname
miniroot# sed -e 's/miniroot/ubuntu/' < /etc/hosts > /mnt/ubuntu/etc/hosts  
```

新增用户帐户（帐户和密码均是 "ubuntu"）:  

```
miniroot# chroot /mnt/ubuntu useradd -G sudo -m -s /bin/bash ubuntu
miniroot# echo ubuntu:ubuntu | chroot /mnt/ubuntu chpasswd  
```

* 安装必须的包
```
miniroot# mount -t proc none /mnt/ubuntu/proc
miniroot# mount -t devtmpfs none /mnt/ubuntu/dev
miniroot# cp /etc/resolv.conf /mnt/ubuntu/etc/
miniroot# chroot /mnt/ubuntu /bin/bash
root@miniroot:/# apt-get update
root@miniroot:/# apt-get install --no-install-recommends sudo iproute net-tools isc-dhcp-client
root@miniroot:/# exit
miniroot# rm /mnt/ubuntu/etc/resolv.conf
miniroot# umount /mnt/ubuntu/proc
miniroot# umount /mnt/ubuntu/dev  
```

* 启动 Ubuntu  

```
miniroot# boot /mnt:/ubuntu /lib/systemd/systemd
```

提示：如果根设备没有挂载，可以将冒号前的挂载目录替换成根设备文件，miniroot 会自动挂载：  

```
miniroot# boot /dev/sda1:/ubuntu /lib/systemd/systemd  
```

### 初始配置  

* 串口登录 Ubuntu 
 
```
Ubuntu 15.04 ubuntu ttyFIQ0

ubuntu login: ubuntu
Password: ubuntu
Last login: Tue May 26 08:11:03 UTC 2015 on ttyFIQ0
Welcome to Ubuntu 15.04 (GNU/Linux 3.10.0 armv7l)

 * Documentation:  https://help.ubuntu.com/
ubuntu@ubuntu:~$ sudo -s
[sudo] password for ubuntu: ubuntu 
root@ubuntu:~#
```

* 设置网络（DHCP）  

```
root@ubuntu:~# echo auto eth0 > /etc/network/interfaces.d/eth0
root@ubuntu:~# echo iface eth0 inet dhcp >> /etc/network/interfaces.d/eth0
root@ubuntu:~# ln -fs ../run/resolvconf/resolv.conf /etc/resolv.conf
root@ubuntu:~# ifup eth0
```

* 更新软件包

```	
root@ubuntu:~# cp /etc/apt/sources.list /etc/apt/sources.list.orig
root@ubuntu:~# sed -i -e 's,^# deb\(.*\)$,deb\1,g' /etc/apt/sources.list
root@ubuntu:~# apt-get update
root@ubuntu:~# apt-get dist-upgrade
```

* 重启

```
root@ubuntu:~# reboot
```

* 进入 miniroot，编辑环境变量，加入 ubuntu 的启动参数：

```
miniroot# editenv
boot=/dev/sda1:/ubuntu
init=/lib/systemd/systemd
autoboot=1
```

* 保存环境变量并重启

```
miniroot# saveenv
miniroot# reboot -f
```

### 安装软件包 

安装 Lubuntu （LXDE）桌面环境：  

```
root@ubuntu:~# apt-get install lubuntu-desktop
```

### 固化系统

将 U 盘卡拔出，插入到主机系统，挂载到 /mnt 目录上。  
查看根文件系统所需空间的大小：

```
sudo du -hs /mnt/ubuntu
```

视情况对 /mnt/ubuntu 目录进行清理，特别是一些日志目录、临时目录等。  
生成空白磁盘映像文件，以生成 1G 大小的根文件系统磁盘映像文件为例： 

```
cd /new/firmware/work/dir/
dd if=/dev/zero of=linuxroot.img bs=1M count=1024
# 格式化成 ext4 文件系统格式，卷标为 linuxroot
mkfs.ext4 -F -L linuxroot -m 0 linuxroot.img
```

挂载，拷贝数据，然后卸载：   

```
mount -o loop linuxroot.img /opt
cp -a /mnt/ubuntu/* /opt/
umount /opt
```

这样 linuxroot.img 就是最终的根文件系统映像文件了。  

## 常见问题

### 如何恢复正常启动

往 misc 分区烧写 misc.img 后，开发板就会从 recovery 分区启动系统，要恢复回 boot 分区启动，有两种方法：

* 下载 [misc_zero.img](https://pan.baidu.com/s/1yKBQXDUhyGwrwb66WvbmDQ) (提取码:3128), 然后烧写到 misc 分区
* 在开发板的 Linux shell 下运行：

```
sudo dd if=/dev/zero of=/dev/block/mtd/by-name/misc bs=16K count=count=3
sudo sync
sudo reboot
```

## 参考链接

[Ubuntu 14.04 LTS with miniroot](http://wiki.radxa.com/Rock/ubuntu)