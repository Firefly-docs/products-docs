### 公版编译
#### HDMI
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

### 显示屏 DM-M10R800 V2 编译
#### MIPI_DSI
```
./FFTools/make.sh -d rk3566-firefly-aiojd4-mipi101_M101014_BE45_A1 -j8 -l rk3566_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_firefly_aiojd4_mipi-userdebug
```

### 显示屏 DM-M10R800 V3S 编译
#### MIPI_DSI
```
./FFTools/make.sh -d rk3566-firefly-aiojd4-mipi101_BSD1218_A101KL68 -j8 -l rk3566_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_firefly_aiojd4_mipi-userdebug
```

### 双目摄像头 CAM-2MS2MF 编译
#### HDMI+CAM-2MS2MF
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts
    index bd3abc55163..d027cb9f63c 100644
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts
    @@ -14,8 +14,8 @@
      * using single camera xc7160 ----> rk3566-firefly-aiojd4-cam-8ms1m.dtsi
      * using dual camera gc2053/gc2093   ----> rk3566-firefly-aiojd4-cam-2ms2m.dtsi
      */
    -#include "rk3566-firefly-aiojd4-cam-8ms1m.dtsi"
    -//#include "rk3566-firefly-aiojd4-cam-2ms2m.dtsi"
    +//#include "rk3566-firefly-aiojd4-cam-8ms1m.dtsi"
    +#include "rk3566-firefly-aiojd4-cam-2ms2m.dtsi"
    ```
* 编译

    ```
    ./FFTools/make.sh -d  -j8 -l 
    ./FFTools/mkupdate/mkupdate.sh -l 
    ```

### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts
    index 2b64894..cf10a09 100644
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dts
    @@ -15,9 +15,9 @@
      * using dual camera gc2053/gc2093   ----> rk3566-firefly-aiojd4-cam-2ms2m.dtsi
      * using rk628d   ----> rk3566-firefly-aiojd4-tf-hdmi-mipi-rk628.dtsi
      */
    -#include "rk3566-firefly-aiojd4-cam-8ms1m.dtsi"
    +//#include "rk3566-firefly-aiojd4-cam-8ms1m.dtsi"
    //#include "rk3566-firefly-aiojd4-cam-2ms2m.dtsi"
    -//#include "rk3566-firefly-aiojd4-tf-hdmi-mipi-rk628.dtsi"
    +#include "rk3566-firefly-aiojd4-tf-hdmi-mipi-rk628.dtsi"
    ```

* 编译

    ```
    ./FFTools/make.sh -d  -j8 -l 
    ./FFTools/mkupdate/mkupdate.sh -l 
    ```

## 手动编译 EC-R3566PC Android 11.0

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 
make ARCH=arm64 BOOT_IMG=../rockdev//boot.img .img -j8
```

**<font color=#ff0000 size=3>注意：进行内核debug的时候，单编译生成boot.img如果出现运行错误，可参考[FAQs](faqs.html#android-sdk-dan-du-bian-yi-sheng-cheng-de-boot-img-yu-dao-wen-ti)</font>**

* 编译 uboot：

```
cd ~/proj/RK356X/u-boot/
./make.sh rk3566
```

* 编译 Android：

```
cd ~/proj/RK356X/
source build/envsetup.sh
lunch 
make installclean
make -j8
./mkimage.sh
```

## 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l 
```
打包完成后将在rockdev/Image-XXX/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。


[FAQ]:faqs.html#jumptofaq_boot.img