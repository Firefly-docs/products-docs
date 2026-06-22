
## Preparation

Compiling Android requires higher configuration requirements for the machine:

* 64 bits CPU.
* 16GB physical memory + swap memory.
* 30GB the free disk space is used for the build, and the source tree takes up an additional approximately 25GB.

The official recommendation of Ubuntu 14.04 operating system, after testing, Ubuntu 12.04 can also compile and run successfully, just need to meet the hardware and software configuration following [http://source.android.com/source/building.html](http://source.android.com/source/building.html)

Initialization of the compilation environment is referred to [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html).

* Install OpenJDK 8:

```
sudo apt-get install openjdk-8-jdk
```

Tip: Install openjdk-8-jdk would changed the default link to the JDK. We can do as follow:

```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

To switch JDK versions. SDK will use the JDK path set internally when the default JDK of the operating system cannot be found. Therefore, to enable the same machine to compile Android 5.1 and previous versions, it is more convenient to remove the link:

```
$ sudo /var/lib/dpkg/info/openjdk-8-jdk:amd64.prerm remove
```

* Ubuntu 12.04 package installation:

```
sudo apt-get install git gnupg flex bison gperf build-essential \
zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
g++-multilib mingw32 tofrodos gcc-multilib ia32-libs \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 \
lzop libssl1.0.0 libssl-dev
```

* Ubuntu 14.04 package installation:

```
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-gplv2-dev gcc-multilib libswitch-perl \
libssl1.0.0 libssl-dev
```