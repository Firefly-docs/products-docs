# 编译 Android9.0 固件

### 下载 Android SDK

由于 Android SDK 源码包比较大,可以通过如下方式获取Android9.0源码包：
[下载链接](http://www.t-firefly.com/doc/download/65.html#other_333)

下载完成后，在解压前先校验下 MD5 码：
```
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.001
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.002

6e1d3969c8a0f643522727ff07800bb5  rk3399pro_firefly_android9.0_20191126.7z.001
a6b8d6a775c3d5ed28f4d41cb210a84d  rk3399pro_firefly_android9.0_20191126.7z.002
```

然后解压：
```
cd ~/proj/
7z x ./rk3399pro_firefly_android9.0_20191126.7z.001 -oAIO-3399Pro
cd ./AIO-3399Pro
git reset --hard
```



以下为从 gitlab 处更新的方法：：
```
#1. 进入SDK根目录
cd ~/proj/AIO-3399Pro

#2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399pro-pie-bundle.git .bundle

#3. 若下载仓库失败，目前bundle仓库占用空间较大，所以同步的时候可能会出现卡住或失败的问题，
#   可以从下方百度云链接下载并解压到SDK根目录，解压指令如下：

7z x rk3399pro-pie-bundle.7z  -r -o. && mv rk3399pro-pie-bundle/ .bundle/

#4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```

也可以到如下地址在线查看源码：
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)

## AIO-3399Pro-JD4 产品编译方法

### 整体编译
#### 公版编译
* HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4 -j8 -l rk3399pro_firefly_aiojd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4-userdebug
```

#### 显示屏 DM-M10R800 编译
* LVDS + HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4-lvds-HSX101H40C -j8 -l rk3399pro_firefly_aiojd4_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4_lvds-userdebug
```

#### 显示屏 DM-M10R800 V2 MIPI屏模组 编译
* MIPI + HDMI
```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aiojd4-mipi_M101014_BE45_A1 -l rk3399pro_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4_mipi-userdebug
```

#### 双目摄像头 SV-TAYSH-TQ 编译
* HDMI + SV-TAYSH-TQ

修改kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4.dtsi
```
        xc7160b: xc7160b@1b {
+                status = "disabled";
                reset-gpios = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
        };

        xc7160f: xc7160f@1b {
+                status = "disabled";
                reset-gpios = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
        };

        XC7022b: XC7022b@1b{
+                status = "okay";
                reset-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
        };

        XC6130b: XC6130b@23{
+                status = "okay";
                reset-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
        };
```
* 编译
```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4 -j8 -l rk3399pro_firefly_aiojd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4-userdebug
```

### 手动编译 AIO-3399Pro-JD4 Android 9.0


编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/AIO-3399Pro-JD4/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399pro_firefly_aiojd4/boot.img rk3399pro-firefly-aiojd4.img
```

* 注意：若进行内核 debug，需要将 resource.img 和 kernel.img 打包进去 boot.img 后对 boot 分区进行烧写才能生效。

* 编译 uboot：

```
cd ~/proj/AIO-3399Pro-JD4/u-boot/
./make.sh rk3399pro
```

* 编译 Android：

```
cd ~/proj/AIO-3399Pro-JD4/
source build/envsetup.sh
lunch rk3399pro_firefly_aiojd4-userdebug
make -j8
./mkimage.sh
```
[下载链接]:http://www.t-firefly.com/doc/download/65.html


## 分区镜像

* boot.img： Android 的 initramfs 映像，包含Android根目录的基础文件系统，它负责初始化和加载系统分区。
* system.img： ext4 文件系统格式的 Android 文件系统分区映像。
* kernel.img： 内核映像。
* resource.img： Resource 映像， 包含启动图片和内核设备树。
* misc.img： misc 分区映像， 负责启动模式的切换和急救模式参数的传递。
* recovery.img： Recovery 模式映像。
* rk3399_loader_v1.12.112.bin： Loader 文件。
* uboot.img： U-Boot 映像文件。
* trust.img： Arm trusted file (ATF) 映像文件。
* parameter.txt： 分区布局和内核命令行。
* vendor.img： TODO
* oem.img： TODO
* baseparameter.img： TODO


## 其他安卓版本
* <font color=#ff0000 size=3>主要维护：</font>

   [《编译 Android9.0 固件》](compile_android9.0_firmware.md)  
* 支持但不维护：

   [《编译 Android8.1 固件》](compile_android8.1_firmware.md)  


