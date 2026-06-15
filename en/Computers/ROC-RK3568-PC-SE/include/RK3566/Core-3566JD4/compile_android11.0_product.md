### Public Compile
#### HDMI
```
./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
```

### Display DM-M10R800 V2 Compile
#### MIPI_DSI
```
./FFTools/make.sh -d rk3566-firefly-aiojd4-mipi101_M101014_BE45_A1 -j8 -l rk3566_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_firefly_aiojd4_mipi-userdebug
```

### Display DM-M10R800 V3S Compile
#### MIPI_DSI
```
./FFTools/make.sh -d rk3566-firefly-aiojd4-mipi101_BSD1218_A101KL68 -j8 -l rk3566_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_firefly_aiojd4_mipi-userdebug
```

### Dual Camera CAM-2MS2MF Compile
#### HDMI+CAM-2MS2MF
* modify dts

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
* Compile

    ```
    ./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
    ```

### HDMI-TO-MIPI_CSI(RK628D)  Compile
* modify dts

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

* Compile

    ```
    ./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
    ```

## Manually compile ROC-RK3568-PC-SE Android 11.0

Configure the environment variables by executing the following command before compilation:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile kernel：

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 firefly_defconfig android-11.config rk356x.config firefly_wifi.config
make ARCH=arm64 BOOT_IMG=BOOT_IMG=../rockdev/Image-rk3568_firefly_roc_pc_se/boot.img rk3568-firefly-roc-pc-se.img -j8
```

**<font color=#ff0000 size=3>Note: When debugging the kernel, if there is a run error in the single compilation of boot.img, please refer to the [FAQs](faqs.html#problems-encountered-in-compiling-boot-img-of-android)</font>**

* Compile uboot：

```
cd ~/proj/RK356X/u-boot/
./make.sh rk3568
```

* Compile Android：

```
cd ~/proj/RK356X/
source build/envsetup.sh
lunch rk3568_firefly_roc_pc_se-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
```
After packaging, it will be in rockdev/Image-XXX/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.


[FAQ]:faqs.html#jumptofaq_boot.img