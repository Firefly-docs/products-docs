# 编译 Android11.0 固件

运行 Android11.0，Core-3568J需要满足如下条件：
* *DDR*至少需要 <font color=red>2GB</font>
* *eMMC*至少需要 <font color=red>16GB</font>

### Android SDK
SDK 源码与 bundle 压缩包均存放在云盘中。
#### 第一步，下载 SDK
* 由于 SDK 较大，可以去下载页面选择云盘下载 [Firefly-RK356X_Android11.0_git](https://www.t-firefly.com/doc/download/103.html#other_449)

* 下载完成后，在解压前先校验下 MD5 码：

    ```
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.001
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.002
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.003
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.004
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.005

    b4c3d014a688d230bb25156a4c5aea26  Firefly-RK356X_Android11.0_git_20210824.7z.001
    1ddfec58d0d69aee6029982fcbe4343c  Firefly-RK356X_Android11.0_git_20210824.7z.002
    d08b16c244545ac68b496e2980d3c6a7  Firefly-RK356X_Android11.0_git_20210824.7z.003
    2b628cc10a55214b8d9a3619673c01c3  Firefly-RK356X_Android11.0_git_20210824.7z.004
    6f7e63955c96ca3c9ba6e4e49d52c90c  Firefly-RK356X_Android11.0_git_20210824.7z.005
    ```

* 然后解压：

<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

```
    $ mkdir ~/proj
    $ mv /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.*  ~/proj
    $ cd ~/proj/
    $ 7z x ./Firefly-RK356X_Android11.0_git_20210824.7z.001 -oRK356X_Android11.0
    $ cd ./RK356X_Android11.0
    $ git reset --hard
```

#### 第二步，更新 SDK
下载 SDK 后，从 gitlab 处更新代码的方法：
```
#1. 进入 SDK 根目录
cd ~/proj/RK356X_Android11.0

#2. 下载远程 bundle 仓库
git clone https://gitlab.com/TeeFirefly/rk356x-android11-bundle.git .bundle

#3. 若下载仓库失败，目前 bundle 仓库占用空间较大，所以同步的时候可能会出现卡住或失败的问题，
# 可以从云盘下载 bundle 并解压到 SDK 根目录，解压指令如下：

7z x rk356x-android11-bundle.7z  -r -o. && mv rk356x-android11-bundle/ .bundle/

#4. 更新 SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步 FETCH_HEAD 到 firefly 分支
git rebase FETCH_HEAD
```

随着 SDK 的更新，bundle 也会随之越来越大，可以去下载页面选择云盘下载 [bundle](https://www.t-firefly.com/doc/download/103.html#other_449)。
## Core-3568J 产品编译方法

**<font color=#ff0000 size=3>注意：若是第一次编译该Android Product，请使用公版编译命令进行一次完整编译</font>**

### 公版编译
#### HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
```

### IPC-M10R800-A3568J 编译
```
./FFTools/make.sh -d rk3568-firefly-aioj-ipc-mipi_BSD1218_A101KL68 -j8 -l rk3568_firefly_aioj_ipc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_ipc-userdebug
```

### 显示屏 DM-M10R800 V2 编译
#### MIPI_DSI0+HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_mipi-userdebug
```

### 显示屏 DM-M10R800 V3S 编译
#### MIPI_DSI0+HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj-mipi101_BSD1218_A101KL68 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_mipi-userdebug
```

### 双目摄像头 CAM-2MS2MF 编译
#### HDMI+CAM-2MS2MF
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
    index 7e2a8b2..14fa027 100755
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
    @@ -7,6 +7,15 @@
    - #include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
    +//#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
    - //#include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
    + #include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
    ```
* 编译

    ```
    ./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
    ```

### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
    index 14fa027..5bc904f 100755
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
    @@ -13,9 +13,9 @@
    * using dual camera gc2053/gc2093   ----> rk3568-firefly-aioj-cam-2ms2m.dtsi
    * using hdmi-in module rk628d   ----> rk3568-firefly-aioj-tf-hdmi-mipi-rk628.dtsi
    */
    -#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
    +//#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
    //#include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
    -//#include "rk3568-firefly-aioj-tf-hdmi-mipi-rk628.dtsi"
    +#include "rk3568-firefly-aioj-tf-hdmi-mipi-rk628.dtsi"
    ```

* 编译

    ```
    ./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
    ```
<font color="red">注意：AIO-3568J 硬件版本为V1.3及往后版本方可支持使用HDMI TO MIPI_CSI 驱动板。</font>

## 手动编译 Core-3568J Android 11.0

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 firefly_defconfig android-11.config rk356x.config firefly_wifi.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3568_firefly_aioj/boot.img rk3568-firefly-aioj.img -j8
```

**<font color=#ff0000 size=3>注意：进行内核debug的时候，单编译生成boot.img如果出现运行错误，可参考[FAQs](faqs.html#android-sdk-dan-du-bian-yi-sheng-cheng-de-boot-img-yu-dao-wen-ti)</font>**

* 编译 uboot：

```
cd ~/proj/RK356X/u-boot/
./make.sh rk3568
```

* 编译 Android：

```
cd ~/proj/RK356X/
source build/envsetup.sh
lunch rk3568_firefly_aioj-userdebug
make installclean
make -j8
./mkimage.sh
```

## 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
```
打包完成后将在rockdev/Image-XXX/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。


