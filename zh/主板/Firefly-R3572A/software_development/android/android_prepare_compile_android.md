# 编译环境搭建

## 准备工作 [step]

编译 Android 固件对主机配置要求较高，编译前确认以下配置：

* 64 位 CPU
* 64 GB 物理内存与交换内存
* 400GB 空闲磁盘空间
* Ubuntu 20.04 或更高版本

<Callout title="注意" type="warn">

SDK 采用交叉编译，需要在 X86_64 电脑上使用，不要将 SDK 下载到开发板上。

</Callout>

## 安装依赖包 [step]

在 Ubuntu 主机上执行以下命令安装编译依赖：

```bash
sudo apt-get update
sudo apt-get install git-core gnupg flex bison build-essential \
zip curl zlib1g-dev libc6-dev-i386 x11proto-core-dev libx11-dev \
lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig \
device-tree-compiler liblz4-tool python3-pyelftools
```

## 编译 FAQ [step]

不同 PC 的系统版本与环境配置存在差异，安装依赖后编译仍可能因为缺少软件包而失败。例如：

```text
OBJCOPY spl/u-boot-spl-nodtb.bin
CAT     spl/u-boot-spl-dtb.bin
COPY    spl/u-boot-spl.bin
CFGCHK  u-boot.cfg
ERROR: No 'dtc', please: apt-get install device-tree-compiler
```

根据报错信息安装对应软件包即可，例如上述提示需要安装 `device-tree-compiler`：

```bash
sudo apt-get install device-tree-compiler
```

环境准备完成后，按 [编译 Android 固件](android_compile_firmware.md) 的步骤继续。