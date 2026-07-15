# Build Ubuntu Rfs

## Use miniroot to Create and Load System
Homepage of miniroot is: [http://androtab.info/miniroot/](http://androtab.info/miniroot/ )
miniroot is a tiny shell environment, used to create and load root file system, such as Ubuntu, Gentoo and Arch Linux, etc. These systems can be reside in root or sub directories on any supported storage device. This means that we can install multiple OS in eMMC Flash, TF card or USB disk, and boot from one of them. Very handy to switch OS, no need to modify and flash parameter file (parameter file is used to specify the important partition info of flash storage).  
miniroot needs serial line to debug, please see [Serial Debug](debug.md). Additionally, we need ethernet to download system image when install. Of course, it can also be downloaded first on host PC, then copied to the removable storage device for later use.

## Before You Start

Important! Please backup data first, whether in the development board or removable storage devices, in case of any data lost due to mistaken operations or unforeseen accidents.  

First, make sure the board is already flashed with working images. Then download following image files:

* [misc.img](http://www.t-firefly.com/download/fireprime/firmware/misc.img)
* [linux-boot-miniroot.img](http://www.t-firefly.com/download/fireprime/firmware/linux-boot-miniroot.img)

If Android or dual system is installed, then flash linux-boot-miniroot.img to recovery partition, misc.img to misc misc partition respectively.  
If pure Linux is installed, flash linux-boot-miniroot.img to boot partition.
miniroot will drop you to a shell, after first boot up. You can see the prompt in the serial terminal:
```
miniroot#
```

Then start to configure the network. If it is DHCP:
```
miniroot# udhcpc
```

Otherwise, you need to specify network parameters manually. (Replace 192.168.1.* with the address actually used):
```
miniroot# ip addr add 192.168.1.2/24 broadcast + dev eth0
miniroot# ip link set dev eth0 up
miniroot# ip route add default via 192.168.1.1
miniroot# echo nameserver 192.168.1.1 > /etc/resolv.conf
```

Miniroot supports booting from sub directory. You can have the OS installed in the favorite location, and boot from it. Multiple OS boot support is therefore just a piece of cake.  
Note that in Firefly-RK3128, the debug serial port has pins multiplexed with the TF card interface. Therefore, the TF card must be plugged out before using the debug serial port.  
Below, we'll use the first parition of USB disk to install the OS. Create ext4 filesystem and then mount it to /mnt, then extract Ubuntu files:
```
miniroot# mkfs.ext4 -E nodiscard /dev/sda1
miniroot# mount /dev/sda1 /mnt
```

Please make sure the partition has sufficient free space (4G
and above).

## Download and Extract ubuntu-core

ubuntu-core is the minimum root file system. You can setup desktop or server environment later after install.  
Download and extract to /mnt:
```
miniroot# cd /mnt
miniroot# wget -P /mnt http://cdimage.ubuntu.com/ubuntu-core/releases/15.04/release/ubuntu-core-15.04-core-armhf.tar.gzminiroot# mkdir /mnt/ubuntu
miniroot# tar -xpzf /mnt/ubuntu-core-15.04-core-armhf.tar.gz -C /mnt/ubuntu
```

## Boot Ubuntu

* Set host name
```
miniroot# echo firefly > /mnt/ubuntu/etc/hostname
miniroot# sed -e 's/miniroot/firefly/' < /etc/hosts > /mnt/ubuntu/etc/hosts
```

Or serial console is not convenient, you can create new user account (account and password are both "ubuntu" in this case):
```
miniroot# chroot /mnt/ubuntu useradd -G sudo -m -s /bin/bash ubuntu
miniroot# echo ubuntu:ubuntu | chroot /mnt/ubuntu chpasswd
```

* Install core packages
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

* Boot Ubuntu
```
miniroot# boot /mnt:/ubuntu /lib/systemd/systemd
```

Tips: If root device is not mounted, you can replace the mount directory before the colon with the root device. Miniroot will mount the root device automatically:
```
miniroot# boot /dev/sda1:/ubuntu /lib/systemd/systemd
```

## Initial Configuration

* Login by serial console  
```
Ubuntu 15.04 ubuntu ttyFIQ0
ubuntu login: ubuntu
Password: ubuntu
Last login: Tue May 26 08:11:03 UTC 2015 on ttyFIQ0
Welcome to Ubuntu 15.04 (GNU/Linux 3.10.0 armv7l)
Documentation:  https://help.ubuntu.com/
ubuntu@ubuntu:~$ sudo -s
[sudo] password for ubuntu: ubuntu 
root@ubuntu:~#
```

* Setup network（DHCP）
```
root@ubuntu:~# echo auto eth0 > /etc/network/interfaces.d/eth0
root@ubuntu:~# echo iface eth0 inet dhcp >> /etc/network/interfaces.d/eth0
root@ubuntu:~# ln -fs ../run/resolvconf/resolv.conf /etc/resolv.conf
root@ubuntu:~# ifup eth0
```

* Upgrade packages
```
root@ubuntu:~# cp /etc/apt/sources.list /etc/apt/sources.list.orig
root@ubuntu:~# sed -i -e 's,^# deb\(.*\)$,deb\1,g' /etc/apt/sources.list
root@ubuntu:~# apt-get update
root@ubuntu:~# apt-get dist-upgrade
```

* Reboot
```
root@ubuntu:~# reboot
```

* Enter miniroot, edit the environment variables, and set the boot parameters of Ubuntu:
```
miniroot# editenv
boot=/dev/sda1:/ubuntu
init=/lib/systemd/systemd
autoboot=1
```

* Save the environment variables and reboot
```
miniroot# saveenv
miniroot# reboot -f
```

## Install Packages

Install Lubuntu (LXDE):
```
root@ubuntu:~# apt-get install lubuntu-desktop
```

## Make the RFS

Plug USB disk out, and plug it to the host, mount it to  /mnt.
Check the space needed for the rfs:
```
sudo du -hs /mnt/ubuntu
```

Clean /mnt/ubuntu, especially logging and temporary files and directories.  
Create empty disk image file. For example, create 1GB rfs empty image:  
```
cd /new/firmware/work/dir/
dd if=/dev/zero of=linuxroot.img bs=1M count=1024
# format to ext4 filesystem, labeled as linuxroot
mkfs.ext4 -F -L linuxroot linuxroot.img
```

Mount, transfer data, then unmount:
```
mount -o loop linuxroot.img /opt
cp -a /mnt/ubuntu/* /opt/
umount /opt
```

The final root filesystem linuxroot.img is ready to serve.

## FAQ

### How to go back to normal boot

If you flash misc.img to the misc partition, the board will boot the system in recovery partition. To revert to normal boot, there are two ways: 

* Download misc_zero.img, and flash to misc partition.  
* Under a Linux shell in the board, run:  
```
sudo dd if=/dev/zero of=/dev/block/mtd/by-name/misc bs=16K count=3
sudo sync
sudo reboot
```

### Reference

[Ubuntu 14.04 LTS with miniroot](http://androtab.info/radxa_rock/ubuntu/)