## 编译 Yocto 固件

### 获取SDK

```bash
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_yocto_kirkstone_release.xml
.repo/repo/repo sync -c
```

### 编译

#### 选择映像
Yocto 项目提供了一些可用于不 layer 的映像。下表列出目前支持构建的映像和相关配方。

| 映像名字  | 描述 | 提供的layer |
| :--------: | :-------: | :-------: |
| core-image-minimal | A small image that only allows a device to boot | Poky |
| core-image-minimal-xfce | A XFCE minimal demo image | meta-openembedded/meta-xfce |
| core-image-sato | Image with Sato, a mobile environment and visual style for mobile devices. The image supports X11 with a Sato theme, Pimlico applications, and contains terminal, editor, and file manager | Poky |
| core-image-weston | A very basic Wayland image with a terminal | Poky |
| core-image-x11 | A very basic X11 image with a terminal | Poky |

### 编译映像文件

**使用 bitbake 命令构建的过程需要保证网络连接正常，如果是中国内陆客户需要保证能 ping 通外网**

进入目录 <path/to/yocto/poky> ，按顺序执行如下命令

```bash
# Install the required environment packages
# sudo apt install zstd
source oe-init-build-env

# 添加 layer（只需要执行一次）
bitbake-layers add-layer ../../meta-openembedded/meta-oe
bitbake-layers add-layer ../../meta-openembedded/meta-python
bitbake-layers add-layer ../../meta-openembedded/meta-networking
bitbake-layers add-layer ../../meta-openembedded/meta-multimedia
bitbake-layers add-layer ../../meta-openembedded/meta-gnome
bitbake-layers add-layer ../../meta-openembedded/meta-xfce
bitbake-layers add-layer ../../meta-clang
bitbake-layers add-layer ../../meta-browser/meta-chromium
bitbake-layers add-layer ../../meta-rockchip
```



选择其中之一命令来编译完整 core-image recipes 。以下是基于 x11 的 core-image 。

```bash
MACHINE= bitbake core-image-minimal
MACHINE= bitbake core-image-minimal-xfce
MACHINE= bitbake core-image-x11
MACHINE= bitbake core-image-sato
```



以下是基于 wayland 的 core-image 。需要在 ```/path/to/yocto/meta-rockchip/conf/machine/include/display.conf``` 修改 DISPLAY_PLATFORM 为 wayland 。修改如下：

```
DISPLAY_PLATFORM ?= "wayland"
# DISPLAY_PLATFORM ?= "x11"
```

完成上述修改后，执行命令编译 core-image-weston

```bash
MACHINE= bitbake core-image-weston
```

注意：如果在已经进行了完整编译一次 core-image 的基础上，需要更换编译的 core-image recipes 。需要将当前编译过 core-image 的清理掉，再开始编译新的 core-image 。

例如：当前编译的是 core-image-minimal 。需要更换成 core-image-sato 。

```bash
MACHINE= bitbake core-image-minimal -c clean
MACHINE= bitbake core-image-sato
```



如果想单独编译部分 recipes 可以参考以下内容：

```bash
# kernel
MACHINE= bitbake linux-rockchip
        
# u-boot
MACHINE= bitbake u-boot-rockchip
        
# rkmpp
MACHINE= bitbake rockchip-mpp
        
# rockchip-librga
MACHINE= bitbake rockchip-librga
        
# 参看更多编译对象
MACHINE= bitbake -s
```



### 调整编译速度

修改文件```/path/to/yocto/meta-rockchip/conf/machine/firefly-rk356x-kernel5-10.conf``` 中的 BB_NUMBER_THREADS 和 PARALLEL_MAKE 变量配置。若线程数量设置过大可能会导致机器内存不足，导致编译失败。请根据编译机器的配置来设置编译速度。

```
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j 4"
```

- [BB_NUMBER_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_THREADS): The maximum number of threads BitBake simultaneously executes.
- [BB_NUMBER_PARSE_THREADS](https://docs.yoctoproject.org/ref-manual/variables.html#term-BB_NUMBER_PARSE_THREADS): The number of threads BitBake uses during parsing.
- [PARALLEL_MAKE](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKE): Extra options passed to the `make` command during the [do_compile](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-compile) task in order to specify parallel compilation on the local build host.
- [PARALLEL_MAKEINST](https://docs.yoctoproject.org/ref-manual/variables.html#term-PARALLEL_MAKEINST): Extra options passed to the `make` command during the [do_install](https://docs.yoctoproject.org/ref-manual/tasks.html#ref-tasks-install) task in order to specify parallel installation on the local build host.

### 更多 bitbake 选项

从根本上说，BitBake 是一个通用任务执行引擎，它允许 shell 和 Python 任务高效并行运行，同时在复杂的任务间依赖约束下工作。 BitBake 的主要用户之一，OpenEmbedded，利用这个核心并使用面向任务的方法构建嵌入式 Linux 软件堆栈。更多详细使用方法请查看[《bitbake-user-manual》](https://docs.yoctoproject.org/bitbake/2.0/bitbake-user-manual/bitbake-user-manual-intro.html#)。

```bash
MACHINE= bitbake <target> <paramater>
# e.g
MACHINE= bitbake u-boot-rockchip -c clean
MACHINE= bitbake u-boot-rockchip
```

| Bitbake paramater  | 描述 |
| :--------: | :-------: |
| -c fetch | 拉取目标所需要的代码 |
| -c clean | 清除目标的输出文件 |
| -c cleanall | 删除目标所有输出文件、共享高速缓存（shared state cache）和源代码 |
| -c compile -f | 使用此选项可在部署映像后强制重新编译，但不建议使用，除非 Yocto Project 不知道目标代码已经发生改变 |
| -c listtasks | 列出目标定义的所有 target |



### 分区固件烧写

编译生成的固件位于目录```<path/to/yocto>/build/tmp/deploy/images/<board>/```

```bash
$ sudo upgrade_tool di -boot boot.img
$ sudo upgrade_tool di -uboot uboot.img
$ sudo upgrade_tool di -misc misc.img
$ sudo upgrade_tool di -recovery recovery.img
```

* 分区烧写适用于调试阶段，固件验证请使用下文的统一固件烧写
* rootfs 不支持单独烧写，需要打包完整固件再烧写

### 统一固件烧写

编译生成的固件位于目录```<path/to/yocto>/build/tmp/deploy/images/<board>/```，待下载的文件为.wic与update.img，进入loader模式后执行如下命令：

```bash
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```
* **固件默认登录账号为：root，密码为：firefly 。固件含有普通用户账号名称为：firefly ，密码为：firefly 。**

注意：如果客户在 Windows PC 上开发，使用 RKdevtool 直接烧录 update.img 即可，**不需要烧录 `<IMAGE NAME>.wic`**。但是要注意一点是 update.img 是一个链接文件，实际得选择链接文件所指向的实际文件。

### 相关概述
Yocto Project 是一个专注于嵌入式 Linux® 操作系统开发的开源协作项目，它提供灵活的工具集和开发环境，允许全球的嵌入式设备开发人员通过共享技术，软件堆栈，配置和用于创建这些定制的Linux映像的最佳实践进行协作。有关 Yocto 项目的更多信息，请参阅 Yocto Project 官网：[www.yoctoproject.org/](www.yoctoproject.org/)。 Yocto Project 官网上有 [Yocto Project Reference Manual](https://docs.yoctoproject.org/current/ref-manual/index.html) 和 [Yocto Project Overview](https://docs.yoctoproject.org/) 等相关文档详细描述了如何构建系统。


### Yocto Project Release layer 介绍

| layer  | 路径 | 优先级(数字越大优先级越高) | 描述 |
| :--------: | :-------: | :-------: | :-------: |
| meta-oe | meta-openembedded/meta-oe | 6 | contains a large amount of additional recipes |
| meta-python | meta-openembedded/meta-python |  7 |  Provide Python recipes|
| meta-qt5 | meta-qt5 | 7 | Provides QT5 recipes |
| meta-clang | meta-clang |  7 |  clang compiler |
| meta-rockchip | meta-rockchip |  9 |  Rockchip board level support available|
| meta | meta |  5 |  Contains the OpenEmbedded-Core metadata|
| meta-poky | meta-poky |  5 |  Holds the configuration for the Poky reference distribution|
| meta-yocto-bsp | meta-yocto-bsp |  5 |  Configuration for the Yocto Project reference hardware board support package.|
| meta-chromium | meta-chromium |  7 |  Provide chromium browser recipe|