# Compile Android firmware

<CodeBlockTabs defaultValue="a11">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="a11">Android 11</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="a11">

In order to run Android 11.0 normally, ROC-RK3568-PC-SE needs to meet the following conditions:
- *DDR* requires at least **2 GB**
- *eMMC* requires at least **16 GB**

## Android SDK

## Download SDK [step]

- Due to the larger SDK, you can choose the cloud disk to download [Firefly-RK356X_Android11.0_git](https://community.t-firefly.com/en/doc/download/180) from the download page.

- After downloading, verify the MD5 code:

    ```text
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

- After confirming that it is correct, you can unzip:

<Callout title="Attention" type="warn">

To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-English directories.

</Callout>

```bash
    $ mkdir ~/proj
    $ mv /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.* ~/proj
    $ cd ~/proj/
    $ 7z x ./Firefly-RK356X_Android11.0_git_20210824.7z.001 -oRK356X_Android11.0
    $ cd ./RK356X_Android11.0
    $ git reset --hard
```

## Update SDK [step]

Note: Be sure to update the remote warehouse after decompression. The following is how to update from GitLab:

```bash
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

Google Drive [bundle download](https://drive.google.com/drive/folders/1JHuD__-tQVzMILIRv8KGjTRKBDHxrd_A?usp=sharing).
## ROC-RK3568-PC-SE product compilation method [step]

**<font color=#ff0000 size=3>Note: if the Android product is compiled for the first time, please use the public compilation command for a complete compilation </font>**

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">Full compilation</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">Step by step</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA Compilation</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">

### Compile configuration overview

<Callout title="Module info" type="info">

Modules used in this section: displays DM-M10R800 V2 / DM-M10R800 V3S, dual camera CAM-2MS2MF, HDMI-TO-MIPI_CSI driver board (RK628D).

</Callout>

| Configuration | Module | DTS (`-d` parameter) | lunch (`-l` parameter) |
| --- | --- | --- | --- |
| Public Compile (HDMI) | — | `rk3568-firefly-roc-pc-se` | `rk3568_firefly_roc_pc_se-userdebug` |
| Display DM-M10R800 V2 Compile (MIPI_DSI0+HDMI) | DM-M10R800 V2 | `rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1` | `rk3568_firefly_roc_pc-userdebug` |
| Display DM-M10R800 V3S Compile (MIPI_DSI0+HDMI) | DM-M10R800 V3S | `rk3568-firefly-roc-pc-se-mipi101_BSD1218_A101KL68` | `rk3568_firefly_roc_pc-userdebug` |
| Dual Camera CAM-2MS2MF Compile (HDMI+CAM-2MS2MF) | CAM-2MS2MF | `rk3568-firefly-roc-pc-se` (modify dts first) | `rk3568_firefly_roc_pc_se-userdebug` |
| HDMI-TO-MIPI_CSI(RK628D) Compile | RK628D | `rk3568-firefly-roc-pc-se` (modify dts first) | `rk3568_firefly_roc_pc_se-userdebug` |

### Public Compile [step]
**HDMI**

```bash
./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
```

### Display Compile [step]

**DM-M10R800 V2**

**MIPI_DSI0+HDMI**

```bash
./FFTools/make.sh -d rk3568-firefly-roc-pc-se-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_roc_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc-userdebug
```

**DM-M10R800 V3S**

**MIPI_DSI0+HDMI**

```bash
./FFTools/make.sh -d rk3568-firefly-roc-pc-se-mipi101_BSD1218_A101KL68 -j8 -l rk3568_firefly_roc_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc-userdebug
```

### Camera Compile [step]
**HDMI+CAM-2MS2MF**
- modify dts

    ```diff
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

- compile

    ```bash
    ./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
    ```

### RK628D Compile [step]
- modify dts

    ```diff
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

- compile

    ```bash
    ./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
    ```

After the compilation completes, the unified firmware update.img will be generated in the rockdev/Image-XXX/ directory.


  </CodeBlockTab>

  <CodeBlockTab value="partial">

## Manually compile ROC-RK3568-PC-SE Android 11.0 [step]

Before compiling, execute the following command to configure environment variables:

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

- Compile the kernel:

```bash
cd ~/proj/RK356X/kernel/
make ARCH=arm64 firefly_defconfig android-11.config rk356x.config firefly_wifi.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3568_firefly_roc_pc_se/boot.img rk3568-firefly-roc-pc-se.img -j8
```

<Callout title="Note" type="info">

When debugging the kernel, if there is a run error in the single compilation of boot.img, please refer to the [FAQs](faqs.html#problems-encountered-in-compiling-boot-img-of-android).

</Callout>

- Compile uboot:

```bash
cd ~/proj/RK356X/u-boot/
./make.sh rk3568
```

- Compile Android:

```bash
cd ~/proj/RK356X/
source build/envsetup.sh
lunch rk3568_firefly_roc_pc_se-userdebug
make installclean
make -j8
./mkimage.sh
```

## Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command:

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
```
After packaging, it will be in rockdev/Image-XXX/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create update.img and store it in rockdev \ Image directory.

  </CodeBlockTab>

  <CodeBlockTab value="ota">
## OTA Compilation

### Firefly full OTA compilation and upgrade [step]
**Add the -o parameter for OTA full compilation**

```bash
./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug -o
```

**Generate the full package path**

```text
out/target/product/rk3568_firefly_roc_pc_se/rk3568_firefly_roc_pc_se-ota-eng.xxx.zip
```

**Generate the base material package path**

```text
out/target/product/rk3568_firefly_roc_pc_se/obj/PACKAGING/target_files_intermediates/rk3568_firefly_roc_pc_se-target_files-eng.xxx.zip
```
Copy the base material package and rename it, mainly for subcontracting upgrades.

```bash
cp out/target/product/rk3568_firefly_roc_pc_se/obj/PACKAGING/target_files_intermediates/rk3568_firefly_roc_pc_se-target_files-eng.xxx.zip ~/ota/v1.zip
```

**Package compiled firmware and upgrades**

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_roc_pc_se-userdebug
```

In the `rockdev/Image-rk3568_firefly_roc_pc_se/` directory, find the corresponding firmware and burn it to the machine, then the machine can carry out the full package upgrade and the difference package upgrade. Put `update.zip` into /sdcard/ and restart the full package as prompted.

### Generate the incremental OTA package [step]

**Generate a new base material package**

The v1.zip basic material package has been made, and another v2.zip material package needs to be made to generate the difference package. You need to modify the kernel DTS code or Android code, and then run the command again.

```bash
./FFTools/make.sh -d rk3568-firefly-roc-pc-se -j8 -l rk3568_firefly_roc_pc_se-userdebug -o
```

Regenerate a new base material package that we can copy.

```bash
cp out/target/product/rk3568_firefly_roc_pc_se/obj/PACKAGING/target_files_intermediates/rk3568_firefly_roc_pc_se-target_files-eng.xxx.zip  ~/ota/v2.zip
```

**Generate the incremental OTA package command**

```bash
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```

**Upgrade OTA incremental OTA package**

Put update.zip to the machine sdcard directory reboot, it will automatically recognize, and then click the steps to complete the upgrade.

### OTA server setup

Refer to [OTA Documentation]() to set up the server.
  </CodeBlockTab>
</CodeBlockTabs>

  </CodeBlockTab>
</CodeBlockTabs>
