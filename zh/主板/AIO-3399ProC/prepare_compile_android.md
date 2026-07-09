## 编译环境搭建


### 准备工作

编译 Android 对机器的配置要求较高：

* 64 位 CPU
* 16GB 物理内存+交换内存
* 30GB 空闲的磁盘空间用于构建，源码树另外占用大约 25GB, 如果要编译源码则还需要 150GB 的磁盘空间。

RK官方推荐 Ubuntu 14.04 操作系统，不过该系统比较老且不再维护了，所以这里推荐使用 Ubuntu 16.04或18.04，只需要满足 [http://source.android.com/source/building.html](http://source.android.com/source/building.html) 里的软硬件配置即可。

编译环境的初始化可参考 [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html) 。

* 安装 OpenJDK 8:

```
sudo apt-get install openjdk-8-jdk
```

提示：安装 openjdk-8-jdk，会更改 JDK 的默认链接，这时可用：

```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

来切换 JDK 版本。SDK 在找不到操作系统默认 JDK 的时候会使用内部设定的 JDK 路径，因此，为了让同一台机器可以编译 Android 5.1 及之前的版本，去掉链接更方便：

```
$ sudo /var/lib/dpkg/info/openjdk-8-jdk:amd64.prerm remove
```

* Ubuntu16.04或18.04 软件包安装：

```
sudo apt-get update

sudo apt-get install git gnupg flex bison gperf libsdl1.2-dev \
libesd-java libwxgtk3.0-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-dev gcc-multilib libswitch-perl libssl-dev unzip zip
```
