## 编译环境搭建

### 准备工作

编译 Android 对机器的配置要求较高：

* 64 位 CPU
* 64GB 物理内存+交换内存
* 400GB  空闲的磁盘空间


建议使用 Ubuntu 18.04 操作系统或更高版本(Android14 建议使用 Ubuntu 20.04 操作系统或更高版本)



* Ubuntu 16.04 或 18.04 软件包安装参考：

```
sudo apt-get update

sudo apt-get install git gnupg flex bison gperf libsdl1.2-dev \
libesd-java libwxgtk3.0-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-dev gcc-multilib libswitch-perl libssl-dev unzip zip device-tree-compiler \
liblz4-tool python-pyelftools python3-pyelftools -y 
```


### 编译FAQ

由于每个人的PC系统版本和环境配置不一样，安装软件包后编译并不一定都会成功，可能会出现缺少某些软件包而引起的错误，如：

#### Q1

```
 OBJCOPY spl/u-boot-spl-nodtb.bin
  CAT     spl/u-boot-spl-dtb.bin
  COPY    spl/u-boot-spl.bin
  CFGCHK  u-boot.cfg
ERROR: No 'dtc', please: apt-get install device-tree-compiler
```
此时视报错信息去安装缺少的软件包（dtc）即可