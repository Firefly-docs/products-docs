# 编译 Android14.0 固件

## 下载 Android14.0 SDK

SDK 源码与 bundle 压缩包均存放在云盘中。如有需要请联系商务 ： sales@t-firefly.com



### 下载 SDK
* SDK通过邮件的方式获取，把订单号发送到<font color=red>sales@t-firefly.com</font>邮箱并注明需要的SDK名称  
[firefly_rk3588_android14_git_20240822](https://www.t-firefly.com/doc/download/160.html)

* 下载完成后，在解压前先校验下 MD5 码：

    ```
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.001
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.002
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.003
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.004
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.005
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.006
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.007
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.008
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.009
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.010
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.011

    c9cc4863a3ad3b48f3c74d1f061d0438  firefly_rk3588_android14_git_240822.7z.001
    e81083e1b9b435e4e666f7d08dcd1bd6  firefly_rk3588_android14_git_240822.7z.002
    49b7aace84ab01b9a014b410fa26f42f  firefly_rk3588_android14_git_240822.7z.003
    1632fc98d881d5f4a8ed953afc9c73b1  firefly_rk3588_android14_git_240822.7z.004
    f581633cefdef6103de584501612e5dd  firefly_rk3588_android14_git_240822.7z.005
    f0e591f54a8969755775642567dabb17  firefly_rk3588_android14_git_240822.7z.006
    ee3d00d688ac0ad2bb43f8fe73f354bd  firefly_rk3588_android14_git_240822.7z.007
    09dc458646cce9f53a792d68e0ece3ec  firefly_rk3588_android14_git_240822.7z.008
    add00c4da466d50782ea5dba279c72d1  firefly_rk3588_android14_git_240822.7z.009
    51438ba104f0a67fd4fa09b5fee0d301  firefly_rk3588_android14_git_240822.7z.010
    b733f1d13c070862b5af6ce71052c020  firefly_rk3588_android14_git_240822.7z.011

    ```

* 然后解压：

    ```
    cd ~/proj/
    7z x ./firefly_rk3588_android14.0_git_20240822.7z.001 -oRK3588_Android14.0
    cd ./RK3588_Android14.0
    git reset --hard
    ```
<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

### 更新 SDK
下载 SDK 后，从 gitlab 处更新代码的方法：
```
#1. 进入 SDK 根目录
cd ~/proj/RK3588_Android14.0

#2. 下载远程 bundle 仓库
git clone https://gitlab.com/T-Firefly/rk3588-android14.0-bundle.git .bundle

#3. bundle仓库会随着更新的资源越多而会越来越大，如果bundle仓库下载速度缓慢或若下载失败，
#   请在资源下载界面选择对应的机器bundle文件进行下载并解压到SDK根目录，解压指令如下:

7z x rk3588-android14.0-bundle.7z  -r -o. && mv rk3588-android14.0-bundle .bundle

#4. 更新 SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步 FETCH_HEAD 到 firefly 分支
git rebase FETCH_HEAD
```

下载页面选择云盘下载 [Android14.0 Bundle](https://www.t-firefly.com/doc/download/160.html#other_871)。
## Core-3588L 产品编译方法

### 整体编译

#### HDMI 固件编译 
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```
#### 显示屏 DM-M10R800 V3S 固件编译：
```
./FFTools/make.sh -d aio-3588l-mipi101-BSD1218-A101KL68 -j8 -l aio_3588l_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l_mipi-userdebug
```
<!--
#### 双目摄像头 CAM-2MS2MF 编译：
##### HDMI+CAM-2MS2MF
* 修改 dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588l.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588l.dts
@@ -9,8 +9,8 @@
 #include "aio-3588l.dtsi"
 //#include "aio-3588l-ext.dtsi"
 
-#include "aio-3588l-cam-8ms1m.dtsi"
-//#include "aio-3588l-cam-2ms2mf.dtsi"
+//#include "aio-3588l-cam-8ms1m.dtsi"
+#include "aio-3588l-cam-2ms2mf.dtsi"
```

* 编译
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```

#### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts
```
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588l.dts
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588l.dts
@@ -9,9 +9,9 @@
 #include "aio-3588l.dtsi"
 //#include "aio-3588l-ext.dtsi"
 
-#include "aio-3588l-cam-8ms1m.dtsi"
+//#include "aio-3588l-cam-8ms1m.dtsi"
 //#include "aio-3588l-cam-2ms2mf.dtsi"
-//#include "aio-3588l-tf-hdmi-mipi-rk628.dtsi"
+#include "aio-3588l-tf-hdmi-mipi-rk628.dtsi"
```
*/

* 编译
```
./FFTools/make.sh -d aio-3588l -j8 -l aio_3588l-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```
-->
### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3588_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config pcie_wifi.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-aio_3588l/boot.img aio-3588l.img -j8

```

* 编译 uboot：

```
cd ~/proj/RK3588_Android14.0/u-boot/
./make.sh rk3588
```

* 编译 Android：

```
cd ~/proj/RK3588_Android14.0/
source build/envsetup.sh
lunch aio_3588l-userdebug
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l aio_3588l-userdebug
```
打包完成后将在rockdev/Image-aio_3588l/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。
## 其他编译说明
### Android14.0 不能直接烧写 kernel.img 和 resource.img
Android14.0的`kernel.img`和`resource.img`包含在`boot.img`中，编译kernel后需要在android根目录下执行`./mkimage.sh`重新打包`boot.img`,然后烧写rockdev/Image-aio_3588l/目录下的`boot.img`。

### 单独编译kernel生成boot.img
编译的原理：在kernel目录下将编译生成的 `kernel.img` 和 `resource.img` 替换到旧的 `boot.img` 中， 所以编译的时候需要用 BOOT_IMG=xxx 参数指定`boot.img`的路径，命令如下：

```
cd ~/proj/RK3588_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-aio_3588l/boot.img aio-3588l.img -j8
```
编译后可以直接烧写kernel目录下的`boot.img`。

## 分区镜像

编译的时候执行 `./mkimage.sh` 会重新打包 `boot.img` 和 `super.img`, 并将其它相关的镜像文件拷贝到目录 rockdev/Image-aio_3588l/ 中。以下列出一般固件用到的镜像文件：

| 固件 | 说明 |
|------|------|
| boot.img | 包含ramdis、kernel、dtb |
| dtbo.img | Device Tree Overlays |
| MiniLoaderAll.bin | 包含一级loader |
| misc.img | 包含recovery-wipe开机标识信息,烧写后会进行recovery |
| parameter.txt | 包含分区信息 |
| recovery.img | 包含recovery-ramdis、kernel、dtb |
| super.img | 包含odm、product、vendor、system、system_ext分区内容 |
| uboot.img | 包含uboot固件 |
| vbmeta.img | 包含avb校验信息,用于AVB校验 |
| update.img | 包含以上需要烧写的img文件,可以用于工具直接烧写整个固件包 |

请参照 [《使用USB线缆升级固件》](upgrade_firmware.md) 一文来烧写分区镜像文件。


## OTA 编译
请参照[《OTA编译》](android_compile_ota_package.md)章节



## 常见问题

### git clone 远程 bundle 仓库失败

Q：[更新SDK](android_compile_Android14.0_firmware.html#geng-xin-sdk)时，git clone 远程 bundle 仓库出错：

```
$ git clone https://gitlab.com/T-Firefly/rk3588-Android14.0-bundle.git .bundle
Cloning into '.bundle'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (18/18), done.
error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: unpack-objects failed
```

A：缓冲区大小不够，需要扩大缓冲区：

```
git config --global https.postBuffer 1048576000
```