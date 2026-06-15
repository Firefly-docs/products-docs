### HDMI+DP 显示编译

* 默认编译HDMI+DP
```
./FFTools/make.sh -j8
./FFTools/mkupdate/mkupdate.sh
```

* EDP7.85编译
```
./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_mid-userdebug
```

* MIPI7.85编译
```
./FFTools/make.sh -j8 -d rk3399-firefly-mipi -l rk3399_firefly_mipi_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_mipi_mid-userdebug
```


## 手动编译 AIO-3399C Android 8.1

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly.img
```

* 编译 uboot：

```
cd ~/proj/AIO-3399C/u-boot/
make rk3399_defconfig
make ARCHV=aarch64 -j8
```

* 编译 Android：

```
cd ~/proj/AIO-3399C/
source build/envsetup.sh
lunch rk3399_firefly_mid-userdebug
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

编译完可以用 Firefly 官方的脚本打包成统一固件，执行如下命令：

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_mid-userdebug
```

根据不同的-l XXX-userdebug参数，打包生成统一固件会存放在不同目录下（rockdev/Image-XXX/）： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 `update.img` 也很简单，将编译生成的文件拷贝到 AndroidTool 的 `rockdev/Image` 目录中，然后运行 `rockdev` 目录下的 `mkupdate.bat` 批处理文件即可创建 `update.img` 并存放到 `rockdev/Image` 目录里。

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