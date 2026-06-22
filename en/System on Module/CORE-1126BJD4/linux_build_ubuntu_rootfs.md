# Create  Ubuntu root file system

## Preparatory work

### Install qemu

Install emulator on Linux PC host:

```
sudo apt-get install qemu-user-static
```

### Download and extract ubuntu-core

The CORE-1126BJD4 Ubuntu root file system is based on Ubuntu base 18.04. Users can go to [ubuntu cdimg](http://cdimage.ubuntu.com/ubuntu-base/releases/18.04/release/) to download, choose to download `ubuntu-base-18.04.1-base-arm64.tar.gz`.

After downloading, create a temporary folder and unzip the root file system:

```
mkdir temp
sudo tar -xpf ubuntu-base-18.04.1-base-arm64.tar.gz -C temp
```

## Modify the root file system

### Preparatory

* Preparatory network:

```
sudo cp -b /etc/resolv.conf temp/etc/resolv.conf
```

* Preparatory qemu:

```
sudo cp /usr/bin/qemu-aarch64-static temp/usr/bin/
```

* Enter the root file system for operation:

```
sudo chroot temp
```

### Update and installation

* Update:

```
apt update
apt upgrade
```

* Install the functionality you need:

```
apt install vim git ....(Add as needed)
```

* Install xubuntu

```
apt install xubuntu-desktop
```

* Errors may occur:

```
E: Unable to locate package xxxx
```

Installation package source is not added to the *`/etc/apt/source.list`*, lead to can't identify the installation package, can add source by oneself, also can use the following given *`source.list`* covered the original *`/etc/apt/source.list`* file:

```
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.

deb http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted

## Uncomment the following two lines to add software from the 'universe'
## repository.
## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://ports.ubuntu.com/ubuntu-ports/ xenial universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial universe
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricted

deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security universe
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security universe
deb http://ports.ubuntu.com/ubuntu-ports/ xenial-security multiverse
deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-security multiverse
```

### Add user and set password

* Add user:

```
useradd -s '/bin/bash' -m -G adm,sudo firefly
```

* Set the password for user:

```
passwd firefly
```

* Set the password for root:

```
passwd root
```

* Modify your root file system can exit:

```
exit
```

## Make the root file system

Make your own root file system, depending on the size of your root file system, and note that the `count` value is modified based on the size of the `temp` folder:

```
dd if=/dev/zero of=linuxroot.img bs=1M count=2048
sudo mkfs.ext4 linuxroot.img
mkdir  rootfs
sudo mount linuxroot.img rootfs/
sudo cp -rfp temp/*  rootfs/
sudo umount rootfs/
e2fsck -p -f linuxroot.img
resize2fs  -M linuxroot.img
```

`linuxroot.img` is the final root file system image file.

## FAQs

### Q: After the root file system is loaded, the size is not normal and does not occupy the entire partition?

**A:** Execute the extended file system command after the system is loaded correctly:

```
 resize2fs /dev/mtd/by-name/linuxroot
```