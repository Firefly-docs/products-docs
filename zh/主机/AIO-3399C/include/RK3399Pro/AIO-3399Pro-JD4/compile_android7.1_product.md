
### HDMI 显示编译

1. 默认编译HDMI+DP
```
./FFTools/make.sh -j8 -d rk3399-firefly -l rk3399_firefly_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_box-userdebug
```

2. EDP7.85编译
```
./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_box-userdebug
```

3. MIPI7.85编译
```
./FFTools/make.sh -j8 -d rk3399-firefly-mipi -l rk3399_firefly_mipi_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_mipi_box-userdebug
```

## 手动编译 AIO-3399C

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

*    编译 kernel：


```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly.img
```

*    编译 U-boot：

```
cd ~/proj/AIO-3399C/u-boot/
make rk3399_box_defconfig
make ARCHV=aarch64 -j8
```

* 编译 Android：

```
cd ~/proj/AIO-3399C/
source build/envsetup.sh
lunch rk3399_firefly_box-userdebug
make -j8
./mkimage.sh
```

## 使用 Firefly 官方脚本编译
* 单独编译 kernel：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -j8
```
## 打包成统一固件

编译完之后，先执行以下命令，然后用Firefly官方的脚本打包成统一的SD卡固件，执行如下命令：

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_box-userdebug
```

根据不同的-l XXX-userdebug参数，打包生成统一固件会存放在不同目录下（rockdev/Image-XXX/）： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 `update.img` 也很简单，将编译生成的文件拷贝到 AndroidTool 的 `rockdev/Image` 目录中，然后运行 `rockdev` 目录下的 `mkupdate.bat` 批处理文件即可创建 `update.img` 并存放到 `rockdev/Image` 目录里。


## 生成tf卡启动的固件

编译完之后，先执行以下命令，然后用Firefly官方的脚本打包成统一的SD卡固件，执行如下命令：
```
./mkimage sdboot
./FFTools/mkupdate/mkupdate.sh update
```

## 烧写分区映像

编译的时候执行 ./mkimage.sh 会重新打包 boot.img 和 system.img, 并将其它相关的映像文件拷贝到目录 rockdev/Image-roc_3399_pc/ 中。以下列出一般固件用到的映像文件：

*    boot.img ：Android 的初始文件映像，负责初始化并加载 system 分区。
*    kernel.img ：内核映像。
*    misc.img ：misc 分区映像，负责启动模式切换和急救模式的参数传递。
*    parameter.txt ：emmc的分区信息
*    recovery.img ：急救模式映像。
*    resource.img ：资源映像，内含开机图片和内核的设备树信息。
*    system.img ：Android 的 system 分区映像，ext4 文件系统格式。
*    trust.img ：休眠唤醒相关的文件
*    rk3399_loader_v1.08.106.bin ：Loader文件
*    uboot.img ：uboot文件

请参照 [《升级固件》](03-upgrade_firmware.md) 一文来烧写分区映像文件。

如果使用的是 Windows 系统，将上述映像文件拷贝到 AndroidTool（Windows 下的固件升级工具）的 `rockdev/Image` 目录中，之后参照升级文档烧写分区映像即可，这>样的好处是使用默认配置即可，不用修改文件的路径。

update.img 方便固件的发布，供终端用户升级系统使用。一般开发时使用分区映像比较方便。