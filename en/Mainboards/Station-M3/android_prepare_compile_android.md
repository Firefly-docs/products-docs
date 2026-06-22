## Compile environment to build

### Ready to compile

Compiling Android requires high machine configuration:

* 64-bit CPU
* 64GB physical memory + swap memory
* 250GB of free disk space 

The official recommendation is the Ubuntu 18.04 operating system. 
The initialization of the compilation environment can refer to [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html) 。


* 18.04 package installation reference:


```
sudo apt-get update
sudo apt-get install git-core gnupg flex bison build-essential zip \
curl zlib1g-dev libc6-dev-i386 x11proto-core-dev libx11-dev lib32z1-dev \
libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
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