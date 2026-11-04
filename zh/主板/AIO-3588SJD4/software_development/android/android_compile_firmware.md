# 编译 Android 固件

<CodeBlockTabs defaultValue="a12">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="a12">Android 12</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="a14">Android 14</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="a12">

## 下载 Android12.0 SDK [step]

SDK 源码与 bundle 压缩包均存放在云盘中。



### 下载 SDK
* SDK通过邮件的方式获取，把订单号发送到<font color=red>sales@t-firefly.com</font>邮箱并注明需要的SDK名称  
[firefly_rk3588_android12_git_20240704](https://community.t-firefly.com/doc/download/172)

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
<Callout title="注意" type="warn">
	不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误。
</Callout>

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

下载页面选择云盘下载 [Android12.0 Bundle](https://community.t-firefly.com/doc/download/172)。

<Callout title="git clone 远程 bundle 仓库失败" type="info">
	更新 SDK 时，git clone 远程 bundle 仓库可能报如下错误：

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

	原因是缓冲区大小不够，执行以下命令扩大缓冲区后重试：

	```
	git config --global https.postBuffer 1048576000
	```
</Callout>
## Core-3588SJD4 产品编译方法 [step]

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">整体编译</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">分步编译</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA 编译</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">
	### 整体编译 [step]

	#### HDMI 固件编译 
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```
	#### 显示屏 DM-M10R800 V2 固件编译：
	```
	./FFTools/make.sh -d aio-3588sjd4-mipi101-M101014-BE45-A1 -j8 -l aio_3588sjd4_mipi-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4_mipi-userdebug
	```
	#### 双目摄像头 CAM-2MS2MF 编译
	##### HDMI+CAM-2MS2MF
	* 修改 dts
	```
	--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	@@ -9,8 +9,9 @@
	 #include "aio-3588sjd4.dtsi"
	-#include "aio-3588sjd4-cam-8ms1m.dtsi"
	#include "aio-3588sjd4-cam-dcphy-8ms1m.dtsi"
	-//#include "aio-3588sjd4-cam-2ms2mf.dtsi"
	+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
	+#include "aio-3588sjd4-cam-2ms2mf.dtsi"
	```

	* 编译
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```

	#### HDMI TO MIPI_CSI(RK628D) 编译
	##### HDMI+CAM-2MS2MF
	* 修改 dts
	```
	--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	@@ -7,9 +7,9 @@
	 /dts-v1/;

	 #include "aio-3588sjd4.dtsi"
	-#include "aio-3588sjd4-cam-8ms1m.dtsi"
	+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
	 //#include "aio-3588sjd4-cam-2ms2mf.dtsi"
	-//#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
	+#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
	 #include "aio-3588sjd4-cam-dcphy-8ms1m.dtsi"

	```
	* 编译
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```
  </CodeBlockTab>

  <CodeBlockTab value="partial">
	### 分步编译 [step]

	1. 编译 kernel：

	```
	cd ~/proj/RK3588_Android12.0/kernel-5.10
	export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
	msk ARCH=arm64   BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8

	```

	2. 编译 uboot：

	```
	cd ~/proj/RK3588_Android12.0/u-boot/
	./make.sh rk3588
	```

	3. 编译 Android：

	```
	cd ~/proj/RK3588_Android12.0/
	source build/envsetup.sh
	lunch aio_3588sjd4-userdebug
	make installclean
	make -j8
	./mkimage.sh
	```

	### 打包成统一固件 update.img [step]

	编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
	```
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```
	打包完成后将在rockdev/Image-aio_3588sjd4/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

	在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。
### 编译产物 [step]

	编译的时候执行 `./mkimage.sh` 会重新打包 `boot.img` 和 `super.img`, 并将其它相关的镜像文件拷贝到目录 rockdev/Image-aio_3588sjd4/ 中。以下列出一般固件用到的镜像文件：

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

	请参照 [《USB升级固件》](../../firmware_upgrade/upgrade_firmware.md) 一文来烧写分区镜像文件。
  </CodeBlockTab>

  <CodeBlockTab value="ota">
	## OTA 编译 

	### Firefly 完整OTA编译和升级
	#### 添加-o 参数进行OTA全局编译
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug -o
	```

	#### 生成完整包路径
	```
	out/target/product/aio_3588sjd4/aio_3588sjd4-ota-eng.xxx.zip 
	```
	#### 生成基础素材包路径
	```
	out/target/product/aio_3588sjd4/obj/PACKAGING/target_files_intermediates/aio_3588sjd4-target_files-eng.xxx.zip
	```
	拷贝基础素材包并重新命名，主要用在差分包升级。
	```
	cp out/target/product/aio_3588sjd4/obj/PACKAGING/target_files_intermediates/aio_3588sjd4-target_files-eng.xxx.zip ~/ota/v1.zip 
	```

	#### 打包编译的固件和升级
	```
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```

	在`rockdev/Image-aio_3588sjd4/` 目录下，找到对应的固件烧录到机器，此后机器就可以进行全量包升级和差分包升级，
	把刚刚编译的完整包命名`aio_3588sjd4-ota-eng.xxx.zip` 为update.zip 放到/sdcard/目录下重启,并按照提示进行全量包升级。

	### OTA 差分包生成
	#### 生成新的基础素材包 
	如上已经制作了v1.zip的基础素材包，需要再做一个v2.zip的素材包才能生成差分包，需修改内核dts相关代码或android的代码，然后重新执行命令
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug -o
	```

	重新生成新的基础素材包我们可以进行拷贝

	```
	 cp out/target/product/aio_3588sjd4/obj/PACKAGING/target_files_intermediates/aio_3588sjd4-target_files-eng.xxx.zip  ~/ota/v2.zip
	```
	#### 生成差分包命令
	```
	out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
	```
	#### 升级OTA差分包 
	把update.zip 放到机器sdcard目录下重启，就会自动识别到，然后按步骤点击升级即可完成。

	### OTA 服务器搭建

	参考[OTA 文档](https://community.t-firefly.com/doc/download/172)搭建服务器。  
  </CodeBlockTab>
</CodeBlockTabs>

<Callout title="其他编译说明" type="info">
	**Android12.0 不能直接烧写 kernel.img 和 resource.img**

	Android12.0的`kernel.img`和`resource.img`包含在`boot.img`中，编译kernel后需要在android根目录下执行`./mkimage.sh`重新打包`boot.img`,然后烧写rockdev/Image-aio_3588sjd4/目录下的`boot.img`。

	**单独编译kernel生成boot.img**

	编译的原理：在kernel目录下将编译生成的 `kernel.img` 和 `resource.img` 替换到旧的 `boot.img` 中， 所以编译的时候需要用 BOOT_IMG=xxx 参数指定`boot.img`的路径，命令如下：

	```
	cd ~/proj/RK3588_Android12.0/kernel-5.10
	export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
	msk ARCH=arm64   BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8
	```
	编译后可以直接烧写kernel目录下的`boot.img`。
</Callout>

  </CodeBlockTab>

  <CodeBlockTab value="a14">

## 下载 Android14.0 SDK [step]

SDK 源码与 bundle 压缩包均存放在云盘中。如有需要请联系商务 ： sales@t-firefly.com



### 下载 SDK
* SDK通过邮件的方式获取，把订单号发送到<font color=red>sales@t-firefly.com</font>邮箱并注明需要的SDK名称  
[firefly_rk3588_android14_git_20240822](https://community.t-firefly.com/doc/download/172)

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
<Callout title="注意" type="warn">
	不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误。
</Callout>

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

下载页面选择云盘下载 [Android14.0 Bundle](https://community.t-firefly.com/doc/download/172)。

<Callout title="git clone 远程 bundle 仓库失败" type="info">
	更新 SDK 时，git clone 远程 bundle 仓库可能报如下错误：

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

	原因是缓冲区大小不够，执行以下命令扩大缓冲区后重试：

	```
	git config --global https.postBuffer 1048576000
	```
</Callout>
## Core-3588SJD4 产品编译方法 [step]

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">整体编译</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">分步编译</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA 编译</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">
	### 整体编译 [step]

	#### HDMI 固件编译 
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```
	#### 显示屏 DM-M10R800 V3S 固件编译：
	```
	./FFTools/make.sh -d aio-3588sjd4-mipi101-BSD1218-A101KL68 -j8 -l aio_3588sjd4_mipi-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4_mipi-userdebug
	```
	<!--
	#### 双目摄像头 CAM-2MS2MF 编译：
	##### HDMI+CAM-2MS2MF
	* 修改 dts
	```
	--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	@@ -9,8 +9,8 @@
	 #include "aio-3588sjd4.dtsi"
	 //#include "aio-3588sjd4-ext.dtsi"

	-#include "aio-3588sjd4-cam-8ms1m.dtsi"
	-//#include "aio-3588sjd4-cam-2ms2mf.dtsi"
	+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
	+#include "aio-3588sjd4-cam-2ms2mf.dtsi"
	```

	* 编译
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```

	#### HDMI TO MIPI_CSI(RK628D) 编译
	* 修改 dts
	```
	--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dts
	@@ -9,9 +9,9 @@
	 #include "aio-3588sjd4.dtsi"
	 //#include "aio-3588sjd4-ext.dtsi"

	-#include "aio-3588sjd4-cam-8ms1m.dtsi"
	+//#include "aio-3588sjd4-cam-8ms1m.dtsi"
	 //#include "aio-3588sjd4-cam-2ms2mf.dtsi"
	-//#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
	+#include "aio-3588sjd4-tf-hdmi-mipi-rk628.dtsi"
	```

	* 编译
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```
	-->
  </CodeBlockTab>

  <CodeBlockTab value="partial">
	### 分步编译 [step]

	1. 编译 kernel：

	```
	cd ~/proj/RK3588_Android14.0/kernel-6.1
	export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64 firefly_defconfig android-14.config pcie_wifi.config
	msk ARCH=arm64 BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8

	```

	2. 编译 uboot：

	```
	cd ~/proj/RK3588_Android14.0/u-boot/
	./make.sh rk3588
	```

	3. 编译 Android：

	```
	cd ~/proj/RK3588_Android14.0/
	source build/envsetup.sh
	lunch aio_3588sjd4-userdebug
	make installclean
	make -j8
	./mkimage.sh
	```

	### 打包成统一固件 update.img [step]

	编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
	```
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```
	打包完成后将在rockdev/Image-aio_3588sjd4/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

	在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。
### 编译产物 [step]

	编译的时候执行 `./mkimage.sh` 会重新打包 `boot.img` 和 `super.img`, 并将其它相关的镜像文件拷贝到目录 rockdev/Image-aio_3588sjd4/ 中。以下列出一般固件用到的镜像文件：

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

	请参照 [《USB升级固件》](../../firmware_upgrade/upgrade_firmware.md) 一文来烧写分区镜像文件。
  </CodeBlockTab>

  <CodeBlockTab value="ota">
	## OTA 编译 

	### Firefly 完整OTA编译和升级
	#### 添加-o 参数进行OTA全局编译
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug -o
	```

	#### 生成完整包路径
	```
	out/target/product/aio_3588sjd4/aio_3588sjd4-ota-eng.xxx.zip 
	```
	#### 生成基础素材包路径
	```
	out/target/product/aio_3588sjd4/obj/PACKAGING/target_files_intermediates/aio_3588sjd4-target_files-eng.xxx.zip
	```
	拷贝基础素材包并重新命名，主要用在差分包升级。
	```
	cp out/target/product/aio_3588sjd4/obj/PACKAGING/target_files_intermediates/aio_3588sjd4-target_files-eng.xxx.zip ~/ota/v1.zip 
	```

	#### 打包编译的固件和升级
	```
	./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
	```

	在`rockdev/Image-aio_3588sjd4/` 目录下，找到对应的固件烧录到机器，此后机器就可以进行全量包升级和差分包升级，
	把刚刚编译的完整包命名`aio_3588sjd4-ota-eng.xxx.zip` 为update.zip 放到/sdcard/目录下重启,并按照提示进行全量包升级。

	### OTA 差分包生成
	#### 生成新的基础素材包 
	如上已经制作了v1.zip的基础素材包，需要再做一个v2.zip的素材包才能生成差分包，需修改内核dts相关代码或android的代码，然后重新执行命令
	```
	./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug -o
	```

	重新生成新的基础素材包我们可以进行拷贝

	```
	 cp out/target/product/aio_3588sjd4/obj/PACKAGING/target_files_intermediates/aio_3588sjd4-target_files-eng.xxx.zip  ~/ota/v2.zip
	```
	#### 生成差分包命令
	```
	out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
	```
	#### 升级OTA差分包 
	把update.zip 放到机器sdcard目录下重启，就会自动识别到，然后按步骤点击升级即可完成。

	### OTA 服务器搭建

	参考[OTA 文档](https://community.t-firefly.com/doc/download/172)搭建服务器。  
  </CodeBlockTab>
</CodeBlockTabs>

<Callout title="其他编译说明" type="info">
	**Android14.0 不能直接烧写 kernel.img 和 resource.img**

	Android14.0的`kernel.img`和`resource.img`包含在`boot.img`中，编译kernel后需要在android根目录下执行`./mkimage.sh`重新打包`boot.img`,然后烧写rockdev/Image-aio_3588sjd4/目录下的`boot.img`。

	**单独编译kernel生成boot.img**

	编译的原理：在kernel目录下将编译生成的 `kernel.img` 和 `resource.img` 替换到旧的 `boot.img` 中， 所以编译的时候需要用 BOOT_IMG=xxx 参数指定`boot.img`的路径，命令如下：

	```
	cd ~/proj/RK3588_Android14.0/kernel-6.1
	export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
	msk ARCH=arm64 BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8
	```
	编译后可以直接烧写kernel目录下的`boot.img`。
</Callout>

  </CodeBlockTab>

</CodeBlockTabs>
