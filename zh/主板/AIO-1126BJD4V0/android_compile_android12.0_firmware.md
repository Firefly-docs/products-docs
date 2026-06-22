# 编译 Android12.0 固件

## 下载 Android12.0 SDK

SDK 源码与 bundle 压缩包均存放在云盘中。



### 下载 SDK
* SDK通过邮件的方式获取，把订单号发送到<font color=red>sales@t-firefly.com</font>邮箱并注明需要的SDK名称  
[firefly_rk3588_android12_git_20240704]()

* 下载完成后，在解压前先校验下 MD5 码：

    ```
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.001
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.002
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.003
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.004
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.005
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.006
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.007

    1ce2a5ab4e3a9f0a4d6d8cd47ca4981d  firefly_rk3588_android12_git_20240704.7z.001
    553fb5ee2cc5623d97bfb19ba299605b  firefly_rk3588_android12_git_20240704.7z.002
    cb9cce227a79356fa57c0781a426fd89  firefly_rk3588_android12_git_20240704.7z.003
    1461a09bacbef0cb3c9a9008e8483e93  firefly_rk3588_android12_git_20240704.7z.004
    d154b39087bbf410a7973da97a6a9ecb  firefly_rk3588_android12_git_20240704.7z.005
    7cfc42443caa57cb4cf474bf3e78b5ce  firefly_rk3588_android12_git_20240704.7z.006
    9ad4374d89212a882dc00541769732f8  firefly_rk3588_android12_git_20240704.7z.007
    
    ```

* 然后解压：

    ```
    cd ~/proj/
    7z x ./firefly_rk3588_android12_git_20240704.7z.001 -oRK3588_Android12.0
    cd ./RK3588_Android12.0
    git reset --hard
    ```
<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

### 更新 SDK
下载 SDK 后，从 gitlab 处更新代码的方法：
```
#1. 进入 SDK 根目录
cd ~/proj/RK3588_Android12.0

#2. 下载远程 bundle 仓库
git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle

#3. bundle仓库会随着更新的资源越多而会越来越大，如果bundle仓库下载速度缓慢或若下载失败，
#   请在资源下载界面选择对应的机器bundle文件进行下载并解压到SDK根目录，解压指令如下:

7z x rk3588-android12.0-bundle.7z  -r -o. && mv rk3588-android12.0-bundle .bundle

#4. 更新 SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步 FETCH_HEAD 到 firefly 分支
git rebase FETCH_HEAD
```

下载页面选择云盘下载 [Android12.0 Bundle]()。
## AIO-1126BJD4V0 产品编译方法

### 整体编译

#### HDMI 固件编译 
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```
#### 显示屏 DM-M10R800 V2 固件编译：
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### 双目摄像头 CAM-2MS2MF 编译：
##### HDMI+CAM-2MS2MF
* 修改 dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
@@ -8,8 +8,8 @@
 
 #include "rk3588-firefly-itx-3588j.dtsi"
-#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
-//#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
+//#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
+#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```

* 编译
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
@@ -7,9 +7,9 @@
 /dts-v1/;
 
 #include "rk3588-firefly-itx-3588j.dtsi"
-#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
+//#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
 //#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
-//#include "rk3588-firefly-itx-3588j-tf-hdmi-mipi-rk628.dtsi"
+#include "rk3588-firefly-itx-3588j-tf-hdmi-mipi-rk628.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```
* 编译
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 
msk ARCH=arm64   BOOT_IMG=../rockdev//boot.img .img -j8

```

* 编译 uboot：

```
cd ~/proj/RK3588_Android12.0/u-boot/
./make.sh rv1126b
```

* 编译 Android：

```
cd ~/proj/RK3588_Android12.0/
source build/envsetup.sh
lunch 
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l 
```
打包完成后将在rockdev// 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。
## 其他编译说明
### Android12.0 不能直接烧写 kernel.img 和 resource.img
Android12.0的`kernel.img`和`resource.img`包含在`boot.img`中，编译kernel后需要在android根目录下执行`./mkimage.sh`重新打包`boot.img`,然后烧写rockdev//目录下的`boot.img`。

### 单独编译kernel生成boot.img
编译的原理：在kernel目录下将编译生成的 `kernel.img` 和 `resource.img` 替换到旧的 `boot.img` 中， 所以编译的时候需要用 BOOT_IMG=xxx 参数指定`boot.img`的路径，命令如下：

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 
msk ARCH=arm64   BOOT_IMG=../rockdev//boot.img .img -j8
```
编译后可以直接烧写kernel目录下的`boot.img`。

## 分区镜像

编译的时候执行 `./mkimage.sh` 会重新打包 `boot.img` 和 `super.img`, 并将其它相关的镜像文件拷贝到目录 rockdev// 中。以下列出一般固件用到的镜像文件：

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

Q：[更新SDK](android_compile_android12.0_firmware.html#geng-xin-sdk)时，git clone 远程 bundle 仓库出错：

```
$ git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle
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