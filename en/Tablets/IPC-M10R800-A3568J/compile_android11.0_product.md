### Public Compile
#### HDMI
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

### Display DM-M10R800 V2 Compile
#### MIPI_DSI0+HDMI
```
./FFTools/make.sh -d rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_roc_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc-userdebug
```

### Display DM-M10R800 V3S Compile
#### MIPI_DSI0+HDMI
```
./FFTools/make.sh -d rk3568-firefly-roc-pc-se-mipi101_BSD1218_A101KL68 -j8 -l rk3568_firefly_roc_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc-userdebug
```

### Dual Camera CAM-2MS2MF Compile
#### HDMI+CAM-2MS2MF
* modify dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts
    index 7e2a8b2..14fa027 100755
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts
    @@ -7,6 +7,15 @@
    - #include "rk3568-firefly-roc-pc-se-cam-8ms1m.dtsi"
    +//#include "rk3568-firefly-roc-pc-se-cam-8ms1m.dtsi"
    - //#include "rk3568-firefly-roc-pc-se-cam-2ms2m.dtsi"
    + #include "rk3568-firefly-roc-pc-se-cam-2ms2m.dtsi"
    ```
* compile

    ```
    ./FFTools/make.sh -d  -j8 -l 
    ./FFTools/mkupdate/mkupdate.sh -l 
    ```

### HDMI-TO-MIPI_CSI(RK628D)  Compile
* modify dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts
    index 9a6d081b..e8e4712 100755
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc-se.dts
    @@ -13,9 +13,9 @@
    * using dual camera gc2053/gc2093   ----> rk3568-firefly-roc-pc-se-cam-2ms2m.dtsi
    * using rk628d   ----> rk3568-firefly-roc-pc-tf-hdmi-mipi-rk628.dtsi
    */
    -#include "rk3568-firefly-roc-pc-se-cam-8ms1m.dtsi"
    +//#include "rk3568-firefly-roc-pc-se-cam-8ms1m.dtsi"
    //#include "rk3568-firefly-roc-pc-se-cam-2ms2m.dtsi"
    -//#include "rk3568-firefly-roc-pc-tf-hdmi-mipi-rk628.dtsi"
    +#include "rk3568-firefly-roc-pc-tf-hdmi-mipi-rk628.dtsi"
    ```

* compile

    ```
    ./FFTools/make.sh -d  -j8 -l 
    ./FFTools/mkupdate/mkupdate.sh -l 
    ```

## Manually compile IPC-M10R800-A3568J Android 11.0

Before compiling, execute the following command to configure environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 
make ARCH=arm64 BOOT_IMG=../rockdev//boot.img .img -j8
```

**<font color=#ff0000 size=3>Note: When debugging the kernel, if there is a run error in the single compilation of boot.img, please refer to the [FAQs](faqs.html#problems-encountered-in-compiling-boot-img-of-android)</font>**

* Compile uboot:

```
cd ~/proj/RK356X/u-boot/
./make.sh rk3568
```

* Compile Android：

```
cd ~/proj/RK356X/
source build/envsetup.sh
lunch 
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l 
```
After packaging, it will be in rockdev/Image-XXX/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.
