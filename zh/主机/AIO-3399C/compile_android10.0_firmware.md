# 编译 Android10.0 固件

## 下载 Android SDK

由于 SDK 较大，可以去下载页面选择云盘下载 **Firefly-RK3399_Android10.0_git_20211222.7z**：

[下载链接](http://www.t-firefly.com/doc/download/page/id/54.html#other_399)  

注：源码与bundle压缩包均存放在云盘中

下载完成后，在解压前先校验下 MD5 码：
```
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.001
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.002
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.003
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.004
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.005

fb41fcdc48b1cf90ecac4a5bb8fafc7a  Firefly-RK3399_Android10.0_git_20211222.7z.001
82d665fb54fb41245d9317312e7b7578  Firefly-RK3399_Android10.0_git_20211222.7z.002
0dd40827976200a6fb61623c9e87fe7c  Firefly-RK3399_Android10.0_git_20211222.7z.003
32324c3afe615394ee4cdf842c53ea8d  Firefly-RK3399_Android10.0_git_20211222.7z.004
9b1656ae07b4fe9ddf7d9ef2dfa2f95a  Firefly-RK3399_Android10.0_git_20211222.7z.005

```

然后解压：
```
cd ~/proj/
7z x ./Firefly-RK3399_Android10.0_git_20211222.7z.001 -ork3399_Android10.0
cd ./rk3399_Android10.0
git reset --hard
```



以下为从 gitlab 处更新的方法：：
```
#1. 进入SDK根目录
cd ~/proj/rk3399_Android10.0

#2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399-android10.0-bundle.git .bundle

#3. 若下载仓库失败，目前bundle仓库占用空间较大，所以同步的时候可能会出现卡住或失败的问题，
#   可以从下方百度云链接下载并解压到SDK根目录，解压指令如下：

7z x rk3399-android10.0-bundle.7z  -r -o. && mv rk3399-android10.0-bundle/ .bundle/

#4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```
## 编译 Android SDK


### 整体编译
#### 公版编译
##### HDMI

###### AIOC AI版
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai -j8 -l rk3399_firefly_aioc_ai-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
```

#### 显示屏 DM-M10R800 编译
##### LVDS+HDMI

###### AIOC AI版
```
./FFTools/make.sh -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -l rk3399_firefly_aioc_ai_lvds-userdebug -j8
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds-userdebug
```

#### 显示屏 DM-M10R800 V2编译
##### MIPI_DSI1+HDMI

###### AIOC AI版
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-mipi101-JDM101014_BC45_A1 -j8 -l rk3399_firefly_aioc_ai_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_mipi-userdebug
```

#### 显示屏 DM-M10R800 V3编译
##### MIPI_DSI1+HDMI

###### AIOC AI版
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-mipi101-BSD1218-A101KL68 -j8 -l rk3399_firefly_aioc_ai_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_mipi-userdebug
```

### 分步编译 AIO-3399C

编译前执行如下命令配置环境变量：
```
source ./FFTools/build.sh
```

* 编译kernel：
```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig android-10.config rk3399.config
AI  版：make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_firefly_aioc_ai/boot.img rk3399-firefly-aioc-ai.img -j8
```

*    编译uboot：
```
cd ~/proj/AIO-3399C/u-boot/
./make.sh rk3399
```

* 编译android：
```
cd ~/proj/AIO-3399C/
AI  版：lunch rk3399_firefly_aioc_ai-userdebug
make -j8
./mkimage.sh
```

### 打包成统一固件 

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
```

根据不同的-l XXX-userdebug参数，打包生成统一固件会存放在不同目录下（rockdev/Image-XXX/）： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。

## 其他编译说明
### Android10.0不能直接烧写kernel.img和resource.img
Android10.0的kernel.img和resource.img包含在boot.img中，更新编译kernel后需要在android根目
录下执行./mkimage.sh重新打包boot.img。打包后烧写rockdev下面的boot.img，可以使用如下方法单
独编译kernel。
#### 单独编译kernel生成boot.img
编译的原理：在kernel目录下将编译生成的 kernel.img 和 resource.img 替换到旧的 boot.img 中，
所以编译的时候需要用 BOOT_IMG=xxx 参数指定boot.img的路径，命令如下：
``` shell
cd kernel
make ARCH=arm64 firefly_defconfig android-10.config
AI  版：make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_firefly_aioc_ai/boot.img rk3399-firefly-aioc-ai.img -j8
```
编译后可以直接烧写kernel目录下的boot.img到机器的boot位置。

## 分区镜像
* boot.img 包含ramdis、kernel、dtb
* dtbo.img Device Tree Overlays 
* kernel.img includekernel，目前无法单独烧写，需要打包到boot.img内烧写
* MiniLoaderAll.bin 包含一级loader
* misc.img 包含recovery-wipe开机标识信息，烧写后会进行recovery
* odm.img 包含android odm，包含在super.img分区内,单独烧写需要用fastboot烧写
* parameter.txt 包含分区信息
* pcba_small_misc.img 包含pcba开机标识信息，烧写后会进入简易版pcba模式
* pcba_whole_misc.img 包含pcba开机标识信息，烧写后会进入完整版pcba模式
* recovery.img 包含recovery-ramdis、kernel、dtb
* resource.img 包含dtb，kernel和uboot阶段的log及uboot充电logo，目前无法单独烧写，需要打包到boot.img内烧写
* super.img 包含odm、vendor、system分区内容
* system.img 包含android system，包含在super.img分区内,单独烧写需要同fastboot烧写
* trust.img 包含BL31、BL32
* uboot.img 包含uboot固件
* vbmeta.img 包含avb校验信息，用于AVB校验
* vendor.img 包含android vendor，包含在super.img分区内,单独烧写需要同fastboot烧写
* update.img 包含以上需要烧写的img文件，可以用于工具直接烧写整个固件包

## 烧写固件

参考：[《升级固件》](03-upgrade_firmware.md) 
## 其他安卓版本
* <font color=#ff0000 size=3>主要维护：</font>

   [《编译 Android7.1 industry 固件》](compile_android7.1_industry_firmware.md)     [《编译 Android10. 固件》](compile_android10.0_firmware.md)  
* 支持但不维护：

   [《编译 Android7.1 固件》](compile_android7.1_firmware.md)     [《编译 Android8.1 固件》](compile_android8.1_firmware.md)  
