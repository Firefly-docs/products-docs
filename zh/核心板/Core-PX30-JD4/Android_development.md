# Android开发

## ADB 使用

### 前言

adb，全称 Android Debug Bridge，是 Android 的命令行调试工具，可以完成多种功能，如跟踪系统日志，上传下载文件，安装应用等。

### 准备连接
支持两种ADB链接方式：USB和网络连接。  
#### USB方式
+ 在开发板上进入选项->开发人员选项，勾上 “USB 调试” 选项
+ AIO-PX30-JD4用Type-C数据线连接设备OTG口和主机(PX30 Type-C 默认只有 device 模式)
  - 设置->Connected devices(已关联的设备)->连接到PC

#### 网络方式
+ 网络ADB  
  - 查看PX30的IP地址，PC端通过网络访问。例:
```
adb connect + IP
adb shell
```

### Windows下的 ADB 安装

首先参照[安装 RK USB 驱动](upgrade_firmware.html#windows)一节安装好驱动。  
然后下载 [adb.zip](https://pan.baidu.com/s/1CpbEy7yFnyRsXPqeekgL9A?pwd=1234)，解压到`C:\adb`以方便调用。
打开命令行窗口，输入：  
```
cd C:\adb
adb shell
```

如果一切正常，就可以进入`adb shell`，在设备上面运行命令。

### Ubuntu 下的 ADB 安装

* 安装 adb 工具：
```
sudo apt-get install android-tools-adb
```

* 加入设备标识：
```
mkdir -p ~/.android
vi ~/.android/adb_usb.ini
# 添加以下一行
0x2207
```

* 加入 udev 规则：
```
sudo vi /etc/udev/rules.d/51-android.rules
# 添加以下一行：
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
```

* 重新插拔 USB 线，或运行以下命令，让 udev 规则生效：
```
sudo udevadm control --reload-rules
sudo udevadm trigger
```

* 重新启动 adb 服务器
```
sudo adb kill-server
adb start-server
```

### 常用 ADB 命令

#### 连接管理

列出所有连接设备及其序列号
```
adb devices
```

如果有多个连接设备，则需要使用序列号来区分：
```
export ANDROID_SERIAL=<设备序列号>
adb shell ls
```
多设备下连接指定设备
```
adb -s 序列号 shell
```

可以通过网络来连接 adb：
```
# 让设备端的 adbd 重启，并在 TCP 端口 5555 处监听
adb tcpip 5555
# 此时可以断开 USB 连接
# 远程连接设备，设备的 IP 地址是 192.168.1.100
adb connect 192.168.1.100:5555
# 断开连接
adb disconnect 192.168.1.100:5555
```

### 调试

#### 获取系统日志 adb logcat

* 用法
```
adb logcat [选项] [应用标签]
```

* 示例
```
# 查看全部日志
adb logcat
# 仅查看部分日志
adb logcat -s WifiStateMachine StateMachine
```

#### 运行命令 adb shell

##### 获取详细运行信息 adb bugreport

`adb bugreport`用于错误报告，里面包含大量有用的信息。

* 示例
```
adb bugreport
# 保存到本地，方便用编辑器查看
adb bugreport >bugreport.txt
```

#### root 权限

如果 TARGET_BUILD_VARIANT 使用的是 userdebug 模式，要获得 root 权限，需要先运行：
```
adb root
```

让 adb 的设备端切换到 root 权限模式，这样 adb remount 等需要 root 权限的命令才会成功。  

### 应用管理

#### 安装应用 adb install

* 用法：
```
adb install [选项] 应用包.apk
```
选项包括：
```
-l forward-lock
-r 重新安装应用，保留原先数据
-s 安装到 SD 卡上，而不是内部存储
```

* 示例：
```
# 安装 facebook.apk
adb install facebook.apk
# 升级 twitter.apk
adb install -r twitter.apk
```

如果安装成功，工具会返回成功提示 "Success"；失败的话，一般是以下几种情况:
* INSTALL_FAILED_ALREADY_EXISTS: 此时需要用 -r 参数来重新安装。
* INSTALL_FAILED_SIGNATURE_ERROR: 应用的签名不一致，可能是发布版和调试版签名不同所致。如果确认 APK 文件签名正常，可以用 [adb uninstall](http://www.t-firefly.com/upload/#adb_uninstall) 命令先卸载旧的应用，然后再安装。
* INSTALL_FAILED_INSUFFICIENT_STORAGE: 存储空间不足，需要检查设备存储情况。

#### 卸载应用 adb uninstall

* 用法：
```
adb uninstall 应用包名称
```

* 示例：
```
adb uninstall com.android.chrome
```
应用包名称可以用以下命令列出：
```
adb shell pm list packages -f
```
运行结果是：  
```
package:/system/app/Bluetooth.apk=com.android.bluetooth
```
前面是 apk 文件，后面则是对应的包名称。

#### 命令行帮助信息 adb help

```
Android Debug Bridge version 1.0.31

 -a                            - directs adb to listen on all interfaces for a connection
 -d                            - directs command to the only connected USB device
                                 returns an error if more than one USB device is present.
 -e                            - directs command to the only running emulator.
                                 returns an error if more than one emulator is running.
 -s <specific device>          - directs command to the device or emulator with the given
                                 serial number or qualifier. Overrides ANDROID_SERIAL
                                 environment variable.
 -p <product name or path>     - simple product name like 'sooner', or
                                 a relative/absolute path to a product
                                 out directory like 'out/target/product/sooner'.
                                 If -p is not specified, the ANDROID_PRODUCT_OUT
                                 environment variable is used, which must
                                 be an absolute path.
 -H                            - Name of adb server host (default: localhost)
 -P                            - Port of adb server (default: 5037)
 devices [-l]                  - list all connected devices
                                 ('-l' will also list device qualifiers)
 connect <host>[:<port>]       - connect to a device via TCP/IP
                                 Port 5555 is used by default if no port number is specified.
 disconnect [<host>[:<port>]]  - disconnect from a TCP/IP device.
                                 Port 5555 is used by default if no port number is specified.
                                 Using this command with no additional arguments
                                 will disconnect from all connected TCP/IP devices.

device commands:
  adb push [-p] <local> <remote>
                               - copy file/dir to device
                                 ('-p' to display the transfer progress)
  adb pull [-p] [-a] <remote> [<local>]
                               - copy file/dir from device
                                 ('-p' to display the transfer progress)
                                 ('-a' means copy timestamp and mode)
  adb sync [ <directory> ]     - copy host->device only if changed
                                 (-l means list but don't copy)
                                 (see 'adb help all')
  adb shell                    - run remote shell interactively
  adb shell <command>          - run remote shell command
  adb emu <command>            - run emulator console command
  adb logcat [ <filter-spec> ] - View device log
  adb forward --list           - list all forward socket connections.
                                 the format is a list of lines with the following format:
                                    <serial> " " <local> " " <remote> "\n"
  adb forward <local> <remote> - forward socket connections
                                 forward specs are one of: 
                                   tcp:<port>
                                   localabstract:<unix domain socket name>
                                   localreserved:<unix domain socket name>
                                   localfilesystem:<unix domain socket name>
                                   dev:<character device name>
                                   jdwp:<process pid> (remote only)
  adb forward --no-rebind <local> <remote>
                               - same as 'adb forward <local> <remote>' but fails
                                 if <local> is already forwarded
  adb forward --remove <local> - remove a specific forward socket connection
  adb forward --remove-all     - remove all forward socket connections
  adb jdwp                     - list PIDs of processes hosting a JDWP transport
  adb install [-l] [-r] [-d] [-s] [--algo <algorithm name> --key <hex-encoded key> --iv <hex-encoded iv>] <file>
                               - push this package file to the device and install it
                                 ('-l' means forward-lock the app)
                                 ('-r' means reinstall the app, keeping its data)
                                 ('-d' means allow version code downgrade)
                                 ('-s' means install on SD card instead of internal storage)
                                 ('--algo', '--key', and '--iv' mean the file is encrypted already)
  adb uninstall [-k] <package> - remove this app package from the device
                                 ('-k' means keep the data and cache directories)
  adb bugreport                - return all information from the device
                                 that should be included in a bug report.

  adb backup [-f <file>] [-apk|-noapk] [-obb|-noobb] [-shared|-noshared] [-all] [-system|-nosystem] [<packages...>]
                               - write an archive of the device's data to <file>.
                                 If no -f option is supplied then the data is written
                                 to "backup.ab" in the current directory.
                                 (-apk|-noapk enable/disable backup of the .apks themselves
                                    in the archive; the default is noapk.)
                                 (-obb|-noobb enable/disable backup of any installed apk expansion
                                    (aka .obb) files associated with each application; the default
                                    is noobb.)
                                 (-shared|-noshared enable/disable backup of the device's
                                    shared storage / SD card contents; the default is noshared.)
                                 (-all means to back up all installed applications)
                                 (-system|-nosystem toggles whether -all automatically includes
                                    system applications; the default is to include system apps)
                                 (<packages...> is the list of applications to be backed up.  If
                                    the -all or -shared flags are passed, then the package
                                    list is optional.  Applications explicitly given on the
                                    command line will be included even if -nosystem would
                                    ordinarily cause them to be omitted.)

  adb restore <file>           - restore device contents from the <file> backup archive

  adb help                     - show this help message
  adb version                  - show version num

scripting:
  adb wait-for-device          - block until device is online
  adb start-server             - ensure that there is a server running
  adb kill-server              - kill the server if it is running
  adb get-state                - prints: offline | bootloader | device
  adb get-serialno             - prints: <serial-number>
  adb get-devpath              - prints: <device-path>
  adb status-window            - continuously print device status for a specified device
  adb remount                  - remounts the /system partition on the device read-write
  adb reboot [bootloader|recovery] - reboots the device, optionally into the bootloader or recovery program
  adb reboot-bootloader        - reboots the device into the bootloader
  adb root                     - restarts the adbd daemon with root permissions
  adb usb                      - restarts the adbd daemon listening on USB
  adb tcpip <port>             - restarts the adbd daemon listening on TCP on the specified port
networking:
  adb ppp <tty> [parameters]   - Run PPP over USB.
 Note: you should not automatically start a PPP connection.
 <tty> refers to the tty for PPP stream. Eg. dev:/dev/omap_csmi_tty1
 [parameters] - Eg. defaultroute debug dump local notty usepeerdns

adb sync notes: adb sync [ <directory> ]
  <localdir> can be interpreted in several ways:

  - If <directory> is not specified, both /system and /data partitions will be updated.

  - If it is "system" or "data", only the corresponding partition
    is updated.

environmental variables:
  ADB_TRACE                    - Print debug information. A comma separated list of the following values
                                 1 or all, adb, sockets, packets, rwx, usb, sync, sysdeps, transport, jdwp
  ANDROID_SERIAL               - The serial number to connect to. -s takes priority over this if given.
  ANDROID_LOG_TAGS             - When used with the logcat option, only these debug tags are printed.
```

## 编译 Android8.1 固件

### 准备工作

编译 Android 对机器的配置要求较高：  

* 64 位 CPU
* 16GB 物理内存+交换内存
* 100GB 空闲的磁盘空间用于构建，源码树另外占用大约 25GB

官方推荐 Ubuntu 14.04 操作系统，经测试，Ubuntu 12.04 也可以编译运行成功，只需要满足 [http://source.android.com/source/building.html](http://source.android.com/source/building.html) 里的软硬件配置即可。  
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

* Ubuntu 12.04 软件包安装：

```
sudo apt-get install git gnupg flex bison gperf build-essential \
zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
g++-multilib mingw32 tofrodos gcc-multilib ia32-libs \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 \
lzop libssl1.0.0 libssl-dev
```

* Ubuntu 14.04 软件包安装：

```
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-gplv2-dev gcc-multilib libswitch-perl \
libssl1.0.0 libssl-dev   
```

### 下载 Android 8.1 SDK  

**Android SDK 源码包比较大,可以通过如下方式获取Android8.1源码包：**
* [[下载链接]](https://community.t-firefly.com/doc/download/63)

下载完成后先验证一下 MD5 码：
```
$md5sum ~/firefly_px30_android8.1_git_20211220.001
$md5sum ~/firefly_px30_android8.1_git_20211220.002
$md5sum ~/firefly_px30_android8.1_git_20211220.003

d8291b39a8be4be2cf7bee4676b53397  firefly_px30_android8.1_git_20211220.001
3f2a0b10ed8b5b85b619e81c71fbfb29  firefly_px30_android8.1_git_20211220.002
7732c2ec7dede98098738b1ffb953993  firefly_px30_android8.1_git_20211220.003

```
确认无误后，就可以解压：
```
cd ~/proj/
7z x ./firefly_px30_android8.1_git_20211220.001 -o AIO-PX30-JD4
cd ./AIO-Px30-JD4
git reset --hard
```
注意：解压后务必要先更新下远程仓库。
以下为从 gitlab 处更新的方法：
```
# 进入SDK根目录
cd ~/proj/AIO-Px30-JD4
# 如果没有拉取远程仓库,需要先拉取对应bundle仓库
git clone https://gitlab.com/TeeFirefly/px30-oreo-bundle.git .bundle
# 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update     
# 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD    
```

### AIO-Px30-JD4产品编译方法
#### lvds显示编译
```
./FFTools/make.sh  -d px30-firefly-aiojd4-lvds -j8 -l px30_evb-userdebug
./FFTools/mkupdate/mkupdate.sh -l px30_evb-userdebug
```

#### 手动编译AIO-Px30-JD4
编译前执行如下命令配置环境变量：
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译kernel：

```
cd ~/proj/AIO-Px30-JD4/android-81/kernel/
make ARCH=arm64 firefly_defconfig
make -j4 ARCH=arm64 px30-firefly-aiojd4-lvds.img
```

* 编译uboot：

```
cd ~/proj/AIO-Px30-JD4/android-81/u-boot/
rm -f *LoaderAll*.bin
make clean
make mrproper
./make.sh evb-px30 -j 4
```

* 编译android：

```
cd ~/proj/AIO-PX30-JD4/android-81/
source FFTools/build.sh
lunch px30_evb-userdebug
make installclean
make -j8
./mkimage.sh

```
### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：

```
./FFTools/mkupdate/mkupdate.sh -l px30_evb-userdebug
```

打包完成后将在rockdev/Image-px30_evb/下生成统一固件：PX30_Android8.1.0_LVDS_xxxxxx.img


在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。
### 烧写分区映像

编译的时候执行 ./mkimage.sh 会重新打包 boot.img 和 system.img, 并将其它相关的映像文件拷贝到目录 rockdev/Image-px30_firefly_aiojd4/ 中。以下列出一般固件用到的映像文件：

*    boot.img ：Android 的初始文件映像，负责初始化并加载 system 分区。
*    kernel.img ：内核映像。
*    misc.img ：misc 分区映像，负责启动模式切换和急救模式的参数传递。
*    parameter.txt ：emmc的分区信息
*    recovery.img ：急救模式映像。
*    resource.img ：资源映像，内含开机图片和内核的设备树信息。
*    system.img ：Android 的 system 分区映像，ext4 文件系统格式。
*    trust.img ：休眠唤醒相关的文件
*    MiniLoaderAll.bin ：Loader文件
*    uboot.img ：uboot文件
*    oem.img ：预置媒体资源及数据包
*    vendor.img ：产品标识和驱动

请参照 [如何升级固件](upgrade_firmware.html) 一文来烧写分区映像文件。

如果使用的是 Windows 系统，将上述映像文件拷贝到 AndroidTool （Windows 下的固件升级工具）的 rockdev\Image 目录中，之后参照升级文档烧写分区映像即可，这样的好处是使用默认配置即可，不用修改文件的路径。

update.img 方便固件的发布，供终端用户升级系统使用。一般开发时使用分区映像比较方便。


## 定制 Android 固件

### 前言

定制 Android 固件，有两种方法：  

* 改源码，然后编译生成固件。  
* 在现有固件的基础上进行裁剪。  

前一种方法，可以从各个层面去定制 Android，自由度大，但对编译环境和技术要求比较高，参见以上《编译 Android8.1 固件》一文。  
现在介绍后一种方法，分为解包、定制和打包三个阶段。主机操作系统为 Linux，采用的工具为开源软件。  

### 固件格式

统一固件 release_update.img，内含启动加载器 loader.img 和真正的固件数据 update.img
```
release_update.img
|- loader.img
`- update.img
```
update.img 是个复合文件，内含多个文件，由 package-file 描述。一个典型的 package-file 为：  
```
# NAME Relative path package-file package-file bootloader Image/MiniLoaderAll.bin parameter       Image/parameter.txt trust           Image/trust.img uboot           Image/uboot.img misc Image/misc.img resource Image/resource.img kernel Image/kernel.img boot            Image/boot.img recovery Image/recovery.img system Image/system.img backup RESERVED #update-script update-script #recover-script recover-script
```

* package-file
* update.img 的打包说明文件，update.img 里也含有一份 package-file。
* Image/MiniLoaderAll.bin 启动加载器，即 bootloader。
* Image/parameter.txt 参数文件，可以设定内核启动参数，里面有重要的分区信息。
* Image/trust.img 是U-Boot作为二级loader 的打包。
* Image/misc.img misc 分区的映像，用来控制 Android 是正常启动，还是进入急救模式（Recovery Mode）。
* Image/kernel.img Android 内核。
* Image/resource.img 资源映像，内有内核开机图片和内核设备树信息（Device Tree Blob)。
* Image/boot.img Android 内核的内存启动盘（initrd)，是内核启动后最先加载的根文件系统，包含重要的初始化动作，一般不需要改动。
* Image/recovery.img Android 急救模式的映像，内含内核和急救模式的根文件系统。
* Image/system.img 对应于 Android 的 /system 分区，是以下的定制对象。

解包，就是提取 release_update.img 里的 update.img, 然后解压出内含 package-file 所声明的多个文件。  
打包，则是个逆过程，将 package-file 将所列的多个文件合成 update.img，加进 loader.img，最终生成 release_update.img 。

### 工具准备
```
git clone https://github.com/TeeFirefly/rk2918_tools.git
cd rk2918_tools
make
sudo cp afptool img_unpack img_maker mkkrnlimg /usr/local/bin
```

### 解包

* 解压 release_update.img
```
$ cd /path/to/your/firmware/dir
$ img_unpack PX30_Android8.1.0_LVDS_xxxxxx.img img
rom version: 6.0.1
build time: 2016-10-27 14:58:18
chip: 33333043
checking md5sum....OK
```

* 解压 update.img
```
$ cd img
$ afptool -unpack update.img update
Check file...OK
------- UNPACK -------
package-file	0x00000800	0x000002AC
Image/MiniLoaderAll.bin	0x00001000	0x0004394E
Image/parameter.txt	0x00045000	0x00000368
Image/trust.img	0x00045800	0x00400000
Image/uboot.img	0x00445800	0x00400000
Image/misc.img	0x00845800	0x0000C000
Image/resource.img	0x00851800	0x00038800
Image/kernel.img	0x0088A000	0x012D2014
Image/boot.img	0x01B5C800	0x0017893C
Image/recovery.img	0x01CD5800	0x00866B8C
Image/system.img	0x0253C800	0x41A9110C
Image/vendor.img	0x43FCE000	0x1206E0A0
Image/oem.img	0x5603C800	0x00058094
RESERVED	0x00000000	0x00000000
UnPack OK!
```

* 查看 update 目录下的文件树

```
$ cd update/
$ tree
.
├── Image
│   ├── boot.img
│   ├── kernel.img
│   ├── MiniLoaderAll.bin
│   ├── misc.img
│   ├── oem.img
│   ├── parameter.txt
│   ├── recovery.img
│   ├── resource.img
│   ├── system.img
│   ├── trust.img
│   ├── uboot.img
│   └── vendor.img
├── loader.img
├── package-file
└── RESERVED

1 directory, 15 files
```

这样，固件就解包成功了，下面就开始定制吧。

#### 打包
首先要检查一下 system.img 的大小，对照 parameter 文件的分区情况(可参考文档[Parameter 文件格式](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf)，作必要的大小调整。   
例如，parameter.txt 文件里的 system 分区大小，可以找到 CMDLINE 一行，然后找到 system 字符串：
```
0x00200000@0x000B0000(system)
```
@ 前面就是分区的大小，单位是 512 字节，这样该 system 分区的大小就是：
```
$ echo $(( 0x00200000 * 512 / 1024 / 1024))M
1024M
```

只要 system.img 的大小不超过 1024M，parameter 文件就不用更改。   
如果分区不用更改，可以直接用烧写工具将新的 system.img 烧写到开发板的 system 分区上做试验。否则，需要制作新固件并烧写后再行测试。   
以下是打包成统一固件 update.img 所需要的步骤： 

* 合成 update.img ：
```
# 当前的目录仍然为 update/ ，内有 package-file, package-file 所列的文件均存在
# 将参数文件拷贝一份到 paramter, 因为 afptool 默认要用到
$ cp Image/parameter.txt parameter
$ afptool -pack . ../update_new.img
------ PACKAGE ------
Add file: ./package-file
Add file: ./Image/MiniLoaderAll.bin
Add file: ./Image/parameter.txt
Add file: ./Image/trust.img
Add file: ./Image/uboot.img
Add file: ./Image/misc.img
Add file: ./Image/resource.img
Add file: ./Image/kernel.img
Add file: ./Image/boot.img
Add file: ./Image/recovery.img
Add file: ./Image/system.img
Add file: ./Image/vendor.img
Add file: ./Image/oem.img
Add file: ./RESERVED
Add CRC...
------ OK ------
Pack OK!
```

* 合成 release_update.img ：
```
$ img_maker -rk33 loader.img update_new.img release_update_new.img
generate image...
append md5sum...
success!
```
release_update_new.img 即为最终生成的可烧写的统一固件文件。

### 常见问题

#### 固件的版本在哪设置
在 parameter 文件中找到下行并修改即可，注意版本号为数字，中间两个点号不能省略。
```
FIRMWARE_VER: 8.1
```