# Compile Android11.0

In order to run Android11.0 normally, Core-3568J needs to meet the following conditions:
* *DDR* requires at least <font color=red>2GB</font>
* *eMMC* requires at least <font color=red>16GB</font>

### Android SDK
#### First, Download SDK
* Due to the larger SDK, we can choose the cloud disk to download **Firefly-RK356X_Android11.0_git** from the download page : [EN_DOW_LINK](http://en.t-firefly.com/doc/download/89.html#other_386) 

* After downloading, verify the MD5 code:

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
* After confirming that it is correct, we can unzip:

<font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

```
    $ mkdir ~/proj
    $ mv /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.* ~/proj
    $ cd ~/proj/
    $ 7z x ./Firefly-RK356X_Android11.0_git_20210824.7z.001 -oRK356X_Android11.0
    $ cd ./RK356X_Android11.0
    $ git reset --hard
```

#### Second, Update SDK
Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/RK356X_Android11.0


2. Download remote bundle repository
git clone https://gitlab.com/TeeFirefly/rk356x-android11-bundle.git .bundle


3. If the download warehouse fails, it may be stuck or failed problems during synchronization. we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk356x-android11-bundle.7z  -r -o. && mv rk356x-android11-bundle/ .bundle/


4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update


5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [bundle download](https://drive.google.com/drive/folders/1JHuD__-tQVzMILIRv8KGjTRKBDHxrd_A?usp=sharing).



## Core-3568J product compilation method

**<font color=#ff0000 size=3>Note: if the Android product is compiled for the first time, please use the public compilation command for a complete compilation </font>**

### Public Compile
#### HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
```

### IPC-M10R800-A3568J Compile
```
./FFTools/make.sh -d rk3568-firefly-aioj-ipc-mipi_BSD1218_A101KL68 -j8 -l rk3568_firefly_aioj_ipc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_ipc-userdebug
```

### Display DM-M10R800 V2 Compile
#### MIPI_DSI0+HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_mipi-userdebug
```

### Display DM-M10R800 V3S Compile
#### MIPI_DSI0+HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj-mipi101_BSD1218_A101KL68 -j8 -l rk3568_firefly_aioj_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_mipi-userdebug
```

### Dual Camera CAM-2MS2MF Compile
#### HDMI+CAM-2MS2MF
* modify dts

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
* Compile

    ```
    ./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
    ```

### HDMI-TO-MIPI_CSI(RK628D)  Compile
* modify dts

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

* Compile

    ```
    ./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
    ```
<font color="red">Note:The AIO-3568J  hardware version must be V1.3 and later to support the use of HDMI-TO-MIPI_CSI drive board.</font>

## Manually compile Core-3568J Android 11.0

Before compiling, execute the following command to configure environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 firefly_defconfig android-11.config rk356x.config firefly_wifi.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3568_firefly_aioj/boot.img rk3568-firefly-aioj.img -j8
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
lunch rk3568_firefly_aioj-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
```
After packaging, it will be in rockdev/Image-XXX/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.



