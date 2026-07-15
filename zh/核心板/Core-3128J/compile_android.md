# 编译 Android 固件

## 准备工作

编译 Android 对机器的配置要求较高：  

* 64 位 CPU
* 16GB 物理内存+交换内存
* 30GB 空闲的磁盘空间用于构建，源码树另外占用大约 25GB

官方推荐 Ubuntu 14.04 操作系统，经测试，Ubuntu 12.04 也可以编译运行成功，只需要满足 [http://source.android.com/source/building.html](http://source.android.com/source/building.html) 里的软硬件配置即可。  
编译环境的初始化可参考 [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html) 。

* 安装 OpenJDK 7：  

```
sudo apt-get install openjdk-7-jdk  
```

提示：安装 openjdk-7-jdk，会更改 JDK 的默认链接，这时可用： 

```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

来切换 JDK 版本。SDK 在找不到操作系统默认 JDK 的时候会使用内部设定的 JDK 路径，因此，为了让同一台机器可以编译 Android 5.1 及之前的版本，去掉链接更方便：

```
$ sudo /var/lib/dpkg/info/openjdk-7-jdk:amd64.prerm remove   
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
  
## 下载 Android SDK  

由于 SDK 比较大，请选择以下云盘下载文件夹`firefly_rk3288_rk3128_android5.1_git_20211216 `：  

* [下载链接](https://www.t-firefly.com/doc/download/page/id/6.html#other_35)


下载完成后先验证一下 MD5 码：  

```
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.002
3d7a59a84e059cd55a68304d47a5462b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
1979bd263b51bec27ec10eceb500731b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.002
```

确认无误后，就可以解压：

```
mkdir -p ~/pro/firefly_rk3128_android5.1
cd ~/pro/firefly_rk3128_android5.1
7z x ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001 -r -o.
git reset --hard 
git checkout fireprime

```

同步方式直接从`gitlab`处更新

```
git pull gitlab fireprime:fireprime
```  

## 编译 Android SDK
### 整体编译

#### 公版编译
##### HDMI
```
./FFTools/make.sh -d rk3128-fireprime -j8 -l rk312x-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk312x-userdebug
```

### 分步编译

* 编译 uboot：
```
cd ~/pro/firefly_rk3128_android5.1/u-boot/
make rk3128_defconfig
make -j8
```

* 编译 kernel：
```
cd ~/pro/firefly_rk3128_android5.1/kernel
make firefly_defconfig
make -j8 rk3128-fireprime.img
```

* 编译 Android:

```
cd ~/pro/firefly_rk3128_android5.1/
source build.sh
lunch rk312x-userdebug
make -j8
./mkimage.sh
```

默认的目标构建变体(TARGET_BUILD_VARIANT)为 userdebug。常用变体有三种，分别是用户(user)、用户调试(userdebug)和工程模式(eng)，其区别如下：  

- user  
    + 仅安装标签为 user 的模块
    +  设定属性 ro.secure=1，打开安全检查功能
    + 设定属性 ro.debuggable=0，关闭应用调试功能
    + 默认关闭 adb 功能
    + 打开 Proguard 混淆器
    + 打开 DEXPREOPT 预先编译优化
- userdebug
    + 安装标签为 user、debug 的模块
    + 设定属性 ro.secure=1，打开安全检查功能
    + 设定属性 ro.debuggable=1，启用应用调试功能
    + 默认打开 adb 功能
    + 打开 Proguard 混淆器
    + 打开 DEXPREOPT 预先编译优化
- eng
    + 安装标签为 user、debug、eng 的模块
    + 设定属性 ro.secure=0，关闭安全检查功能
    + 设定属性 ro.debuggable=1，启用应用调试功能
    + 设定属性 ro.kernel.android.checkjni=1，启用 JNI 调用检查
    + 默认打开 adb 功能
    + 关闭 Proguard 混淆器
    + 关闭 DEXPREOPT 预先编译优化  

如果目标构建变体为 user，则 adb 无法获取 root 权限。  
要选择目标构建变体，可以在 make 命令行加入参数，例如：  
```
make -j8 PRODUCT-rk312x-user
make -j8 PRODUCT-rk312x-userdebug
make -j8 PRODUCT-rk312x-eng  
```

### 打包成统一固件

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：

```
./FFTools/mkupdate/mkupdate.sh -l rk312x-userdebug
```

根据不同的 `-l XXX-userdebug` 参数，打包生成统一固件会存放在不同目录下（rockdev/Image-XXX/）： `product名XXX_XXX_日期XXX.img`

在 Windows 下打包统一固件 `update.img` 也很简单，将编译生成的文件拷贝到 AndroidTool 的 `rockdev\Image` 目录中，然后运行 `rockdev` 目录下的 `mkupdate.bat` 批处理文件即可创建 `update.img` 并存放到 `rockdev\Image` 目录里。

## 烧写分区映像
上一步骤的`./mkimage.sh`会重新打包`boot.img`和`system.img`, 并将其它相关的映像文件拷贝到目录`rockdev/Image-rk312x/`中。以下列出一般固件用到的映像文件：

* boot.img ：Android 的初始文件映像，负责初始化并加载 system 分区。
* kernel.img ：内核映像。
* misc.img ：misc 分区映像，负责启动模式切换和急救模式的参数传递。
* recovery.img ：急救模式映像。
* resource.img ：资源映像，内含开机图片和内核的设备树信息。
* system.img ：Android 的 system 分区映像，ext4 文件系统格式。

请参照 [如何升级固件](upgrade_firmware.md) 一文来烧写分区映像文件。  
如果使用的是 Windows 系统，将上述映像文件拷贝到 AndroidTool （Windows 下的固件升级工具）的 rockdev\Image 目录中，之后参照升级文档烧写分区映像即可，这样的好处是使用默认配置即可，不用修改文件的路径。