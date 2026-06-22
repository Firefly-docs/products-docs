# Compile environment to build

## Ready to compile

Compiling Android requires high machine configuration:

* 64-bit CPU
* 16GB physical memory + swap memory
* 30GB of free disk space is used for construction, and the source tree takes up about 25GBs

The official recommendation is the Ubuntu 14.04 operating system. After testing, Ubuntu 12.04 can also be compiled and run successfully, as long as the software and hardware configurations in the suffice are met.[http://source.android.com/source/building.html](http://source.android.com/source/building.html)

The initialization of the compilation environment can refer to [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html) 。

* Install OpenJDK 7:

```bash
sudo apt-get install openjdk-7-jdk
```

Tip: Installing openjdk-7-jdk will change the default link of the JDK, available at this time:

```bash
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

To switch the JDK version. When the SDK cannot find the default JDK of the operating system, it will use the internal JDK path. Therefore, in order to allow the same machine to compile Android 5.1 and previous versions, it is more convenient to remove the link:

```bash
sudo /var/lib/dpkg/info/openjdk-7-jdk:amd64.prerm remove
```

* Ubuntu 12.04 软件包安装:

```bash
sudo apt-get install git gnupg flex bison gperf build-essential \
zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
g++-multilib mingw32 tofrodos gcc-multilib ia32-libs \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 \
lzop libssl1.0.0 libssl-dev
```

* Ubuntu 14.04 package installation:

```bash
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-gplv2-dev gcc-multilib libswitch-perl \
libssl1.0.0 libssl-dev
```