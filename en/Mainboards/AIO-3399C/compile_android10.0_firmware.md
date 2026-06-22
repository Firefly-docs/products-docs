# Compile Android10.0

### Download Android SDK

Due to the larger SDK, you can choose the cloud disk to download **Firefly-RK3399_Android10.0_git_20211222.7z** from the download page :

[Download link](http://en.t-firefly.com/doc/download/52.html#other_339)

Note: the source code and bundle are stored in the Google Driver


After downloading, verify the MD5 code:
```
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.001
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.002
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.003
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.004
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.005

fb41fcdc48b1cf90ecac4a5bb8fafc7a  Firefly-RK3399_Android10.0_git_20211222.7z.001
82d665fb54fb41245d9317312e7b7578  Firefly-RK3399_Android10.0_git_20211222.7z.002
0dd40827976200a6fb61623c9e87fe7c  Firefly-RK3399_Android10.0_git_20211222.7z.003
32324c3afe615394ee4cdf842c53ea8d  Firefly-RK3399_Android10.0_git_20211222.7z.004
9b1656ae07b4fe9ddf7d9ef2dfa2f95a  Firefly-RK3399_Android10.0_git_20211222.7z.005
```


After confirming that it is correct, you can unzip:
```
cd ~/proj/
7z x ./Firefly-RK3399_Android10.0_git_20211222.7z.001 -ork3399_Android10.0
cd ./rk3399_Android10.0
git reset --hard
```

Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/rk3399_Android10.0


2. Download remote bundle repository
git clone https://gitlab.com/TeeFirefly/rk3399-android10.0-bundle.git .bundle


3. If the download warehouse fails, the current bundle warehouse is about 1.4G, so there may be stuck or failed problems during synchronization. You can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3399-android10.0-bundle.7z  -r -o. && mv rk3399-android10.0-bundle/ .bundle/


4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update


5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

## AIO-3399C product compilation method

### Overall Compilation
#### Public Compile
##### HDMI

###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai -j8 -l rk3399_firefly_aioc_ai-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
```

#### Display DM-M10R800 Compile
##### LVDS+HDMI

###### AIOC AI version
```
./FFTools/make.sh -d rk3399-firefly-aioc-ai-lvds-HSX101H40C -l rk3399_firefly_aioc_ai_lvds-userdebug -j8
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_lvds-userdebug
```

#### Display DM-M10R800 V2 Compile
##### MIPI_DSI1+HDMI

###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-mipi101-JDM101014_BC45_A1 -j8 -l rk3399_firefly_aioc_ai_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_mipi-userdebug
```

#### Display DM-M10R800 V3 Compile
##### MIPI_DSI1+HDMI

###### AIOC AI version
```
./FFTools/make.sh  -d rk3399-firefly-aioc-ai-mipi101-BSD1218-A101KL68 -j8 -l rk3399_firefly_aioc_ai_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai_mipi-userdebug
```

### Compile step by step AIO-3399C

Before compiling, execute the following command to configure environment variables:
```
source ./FFTools/build.sh
```

*    Compile kernel:
```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig android-10.config rk3399.config
AI version: make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_firefly_aioc_ai/boot.img rk3399-firefly-aioc-ai.img -j8
```

*    Compile uboot:
```
cd ~/proj/AIO-3399C/u-boot/
./make.sh rk3399
```

* Compile Android:
```
cd ~/proj/AIO-3399C/
AI version: lunch rk3399_firefly_aioc_ai-userdebug
make -j8
./mkimage.sh
```

### Packaged into unified firmware 

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
AI version:./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aioc_ai-userdebug
```
According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create update.img and store it in rockdev \ Image directory.



### Some Introduction about Compiling 
####  Android 10.0 can't be written directly kernel.img and resource.img 
Android 10.0 kernel.img and resource.img Included in boot.img After the kernel is updated and compiled, it needs to be executed in the Android root directory mkimage.sh Repackage boot.img .After packing, download boot.img  in rockdev directory .You can compile the kernel separately by using the following instruction.
####  Compiling kernel generation separately boot.img 
Principle of compilation: in the kernel directory, the generated kernel.img And resource.img Replace with old boot.img So you need to use boot when compiling  IMG = XXX parameter specification boot.img The command is as follows:
``` shell
cd kernel
make ARCH=arm64 firefly_defconfig android-10.config
AI version: make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_firefly_aioc_ai/boot.img rk3399-firefly-aioc-ai.img -j24
```
After compiling, you can directly download the boot.img To the machine.

### Partition image
* boot.img  include ramdis、kernel、dtb
* dtbo.img  Device Tree Overlays
* kernel.img  Currently, it cannot be burned separately, it needs to be packaged into boot.img to burn and write
* MiniLoaderAll.bin include first loader
* misc.img include recovery-wipe boot flag information ，enter recovery afte v 
* odm.img include android odm，included in super.img partition ,upgrade alone by the fastboot tool 
* parameter.txt include partition information
* pcba_small_misc.img  include pcba boot flag information，enter the simple pcba mode after upgrade 
* pcba_whole_misc.img  include pcba boot flag information，enter the full pcba mode after upgrade
* recovery.img  include recovery-ramdis、kernel、dtb
* resource.img  include dtb，kernel and  uboot phases log and the uboot charging  logo,it should be packed into boot.img,then upgrade it
* super.img   include odm、vendor、system partition content
* system.img  include android system，included in super.img partition ,upgrade alone by the fastboot tool 
* trust.img include BL31、BL32
* uboot.img  include uboot.img
* vbmeta.img include avb，for AVB verify
* vendor.img 包含android vendor，included in super.img partition ,upgrade alone by the fastboot tool 
* update.img include all the img file above，use tools to upgrade the whole firmware package 

### Flash Image 

Reference: [《Flash Image》](03-upgrade_firmware.md)
## Other Android versions
* <font color=#ff0000 size=3>Main maintenance:</font>

   ["Compile Android7.1 Industry firmware"](compile_android7.1_industry_firmware.md)     ["Compile Android10.0"](compile_android10.0_firmware.md)  
* Support but not maintain:

   ["Compile Android7.1 firmware"](compile_android7.1_firmware.md)     ["Compile Android8.1"](compile_android8.1_firmware.md)  
