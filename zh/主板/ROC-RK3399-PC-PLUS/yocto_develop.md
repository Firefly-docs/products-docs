## Yocto 开发

firefly 目前发布支持的Yocto版本有:
* Yocto 2.7.4 : warrior


### I. 准备环境：
100G磁盘可用空间

Git版本要1.8.3.1以上，tar版本要1.28以上，Python3版本要3.5.0以上，gcc版本要5.0以上

执行以下命令安装依赖包：
```
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils
```

### II. 获取SDK

**如果下载了Linux_SDK则无需另外下载Yocto，直接进入linux_sdk/yocot目录即可。**
```
$ repo init --no-clone-bundle --repo-url   https://gitlab.com/firefly-linux/git-repo.git  -u https://gitlab.com/firefly-linux/manifests.git -b master -m yocto_release.xml
$ repo sync -c
```


yocto SDK目录树如下：
```
.
├── bitbake -> poky/bitbake       // OpenEmbedded构建系统时使用的生产工具
├── build                     // 用户配置文件和工程构建输出目录
├── conf
│    └── local.conf -> rk3399/firefly-rk3399.conf  //选择板子配置文件
├── meta-browser              // Web浏览器配方
├── meta-clang                    // 用来构架编译器的LLVM框架系统
├── meta-openembedded         // 用来交叉编译，安装和打包的元数据
├── meta-poky -> poky/meta-poky       // Poky发行版本的配置数据
├── meta-python2              // Python2配方
├── meta-qt5                  // Qt5官方推出的Qt5配方
├── meta-rockchip             // Rockchip层，包含Rockchip芯片BSP相关配方
├── meta-rust                 // Rust与Cargo的OpenEmbedded/Yocto层
├── oe-init-build-env -> poky/oe-init-build-env       // 用于构建OE的环境
├── poky                      // 用来构建Linux的构建系统
└── scripts -> poky/scripts       //　Poky发行版本的脚本文件
```


### III. 编译 Yocto
进入目录<path/to/yocto>，按顺序执行如下命令：

```
source oe-init-build-env

# Select the configuration file of the board. (e.g firefly-rk399.conf)
# cd <path/to/yocto>/build/conf
# ln -sf firefly-rk399.conf local.conf
cd <path/to/yocto>/build
bitbake <target>
```

<target> 公共选项有以下选择

  * core-image-minimal : 一个能够让设备启动的小镜像，它无界面
  * core-image-sato : 一个支持Sato的镜像，它支持带有Sato主题和Pimlico应用程序的X11，还包含终端、编辑器和文件管理器，它是一个基于GNOME Mobile的用户界面环境，使用matchbox作为窗口管理器
  * meta-toolchain：一个可编译出gcc交叉工具链安装程序的选项，生成的文件位于目录<path/to/yocto>/build/tmp/deploy/sdk，主要输出文件为.sh安装文件
  * meta-ide-support：一个用于确保目录<path/to/yocto>/build包含有IDE工具链包的选项，生成的文件为environment-setup-xxxxxx-neon-poky-linux-gnueabi，位于目录<path/to/yocto>/build/tmp，直接用soucre命令运行即可

<target> 其他选项参考文档：[https://www.yoctoproject.org/docs/1.1/poky-ref-manual/poky-ref-manual.html#ref-images](https://www.yoctoproject.org/docs/1.1/poky-ref-manual/poky-ref-manual.html#ref-images)


### IV. 下载固件
编辑此区域
I. Yocto 固件下载：
编译生成的固件位于目录<path/to/yocto>/build/tmp/deploy/<MACHINE>，待下载的文件为.wic与update.img，进入loader模式后执行如下命令：

```
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```