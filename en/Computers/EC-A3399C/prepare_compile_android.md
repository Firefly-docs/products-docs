## Compile environment to build


### Ready to compile

Compiling Android requires high machine configuration:

* 64-bit CPU
* 16GB physical memory + swap memory
* 30GB of free disk space is used for building, the source tree occupies another 25GB, if you want to compile the source code, you need 150GB of disk space.

Rockchip officially recommends the Ubuntu 14.04 operating system, but the system is relatively old and no longer maintained, so Ubuntu 16.04 or 18.04 is recommended here. It only needs to meet [http://source.android.com/source/building.html](http ://source.android.com/source/building.html) can be configured in the software and hardware.

The initialization of the compilation environment can refer to [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html) 。

* Install OpenJDK 8:

```
sudo apt-get install openjdk-8-jdk
```

Tip: Installing openjdk-8-jdk will change the default link of the JDK, available at this time:

```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

To switch the JDK version. When the SDK cannot find the default JDK of the operating system, it will use the internal JDK path. Therefore, in order to allow the same machine to compile Android 5.1 and previous versions, it is more convenient to remove the link:

```
$ sudo /var/lib/dpkg/info/openjdk-8-jdk:amd64.prerm remove
```

* Ubuntu16.04 or 18.04 package installation::
```
sudo apt-get update

sudo apt-get install git gnupg flex bison gperf libsdl1.2-dev \
libesd-java libwxgtk3.0-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-dev gcc-multilib libswitch-perl libssl-dev unzip zip
```
