## 编译环境搭建

### 准备工作

编译 Android 对机器的配置要求较高：

* 64 位 CPU
* 64GB 物理内存+交换内存
* 400GB  空闲的磁盘空间


如需编译 Android 14，您必须使用 Ubuntu 20.04 或更高版本。<br>
如需安装 Ubuntu 20.04 或更高版本所需的软件包，请运行以下命令：
```shell
sudo apt-get update
sudo apt-get install git-core gnupg flex bison build-essential \
zip curl zlib1g-dev libc6-dev-i386 x11proto-core-dev libx11-dev \
lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
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