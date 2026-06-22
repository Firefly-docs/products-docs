## Compile environment to build

### Ready to compile

Compiling Android requires high machine configuration:

* 64-bit CPU
* 16GB physical memory + swap memory
*  of free disk space 

The official recommendation is the Ubuntu 18.04 operating system. 
The initialization of the compilation environment can refer to [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html) 。


* Ubuntu 16.04 and 18.04 package installation reference:

```
sudo apt-get update

sudo apt-get install git gnupg flex bison gperf libsdl1.2-dev \
libesd-java libwxgtk3.0-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-dev gcc-multilib libswitch-perl libssl-dev unzip zip device-tree-compiler \
liblz4-tool python-pyelftools python3-pyelftools -y 
```

### Compilation FAQ
Because everyone's PC system version and environment configuration are different, the compilation after installing the software package is not necessarily successful, and it is inevitable that there will be some errors caused by the lack of some software packages, such as: 

#### Q1

```
 OBJCOPY spl/u-boot-spl-nodtb.bin
  CAT     spl/u-boot-spl-dtb.bin
  COPY    spl/u-boot-spl.bin
  CFGCHK  u-boot.cfg
ERROR: No 'dtc', please: apt-get install device-tree-compiler
```
Don't worry, we could install the lacking software package(dtc) according to the error message, just like you see:

```
apt-get install device-tree-compiler
```