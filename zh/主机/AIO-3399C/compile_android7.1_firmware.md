**下载编译须知：**
目前该Android7.1 版本不做主要维护，请用户编译选择Android7.1 industry 版本<font color="#dd0000">(主要维护)</font>，该版本在工业和平板和盒子等领域的使用>上范围更加广泛，而且性能稳定大批量生产验证过，该版本也作为我司主要维护版本，适用于我司RK3399系统的所有机型。

# 编译 Android7.1 固件


## 下载 Android7.1 SDK

**Android SDK 源码包比较大,可以通过如下方式获取 Android7.1 源码包：**[Android7.1 源码包]

下载完成后先验证一下 MD5 码：

```
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.001
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.002
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.003

90145747b33ad6834cd065b594f6687f  firefly_rk3399_android7.1_git_20211220.7z.001
6f28a1d66b1d168a89bfd3a7923775f8  firefly_rk3399_android7.1_git_20211220.7z.002
dc4d87ba8574900eed84f60f9cbe3646  firefly_rk3399_android7.1_git_20211220.7z.003
```

确认无误后，就可以解压：

```
mkdir -p ~/proj/AIO-3399C
cd ~/proj/AIO-3399C
7z x ~/firefly_rk3399_android7.1_git_20211220.7z.001 -r -o.
git reset --hard
```

* 更新

注意：解压后务必要先更新下远程仓库。以下为从 gitlab 处更新的方法：
```
#1. 进入SDK根目录
cd ~/proj/AIO-3399C

#2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399-nougat-bundle.git .bundle

#3. 若下载仓库失败，则可以从下方百度云下载[bundle压缩包]并解压到SDK根目录，解压指令如下：
7z x rk3399-nougat-bundle-20190612.7z -r -o. && mv rk3399-nougat-bundle/ .bundle/

#4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```

百度云下载[[bundle压缩包]](http://www.t-firefly.com/doc/download/page/id/3.html#other18)



## AIO-3399C 产品编译方法


### HDMI 显示编译

1.AIOC 标准版
```
./FFTools/make.sh  -d rk3399-firefly-aioc -j8 -l rk3399_firefly_aioc_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_box-userdebug
```

2.AIOC AI版
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai -j8 -l rk3399_firefly_aioc_ai_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_box-userdebug
```

### HDMI+lvds编译
1.AIOC 标准版

* 双LVDS
```
./FFTools/make.sh  -d rk3399-firefly-aioc-lvds -j8 -l rk3399_firefly_aioc_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_box-userdebug
```

* 单LVDS
```
./FFTools/make.sh  -d rk3399-firefly-aioc-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_lvds_box-userdebug
```

2.AIOC AI版
* 双LVDS
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-lvds -j8 -l rk3399_firefly_aioc_ai_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds_box-userdebug
```

* 单LVDS
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -j8 -l rk3399_firefly_aioc_ai_lvds_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds_box-userdebug
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
标准版：make -j8 ARCH=arm64 rk3399-firefly-aioc.img
AI  版：make -j8 ARCH=arm64 rk3399-firefly-aioc-ai.img
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
标准版：lunch rk3399_firefly_aioc_box-userdebug
AI  版：lunch rk3399_firefly_aioc_ai_box-userdebug
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
标准版：./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc-userdebug
AI  版：./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
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
[下载链接]: http://www.t-firefly.com/doc/download/page/id/54.html
[Android7.1 源码包]: http://www.t-firefly.com/doc/download/page/id/54.html#other_18
