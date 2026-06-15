### Overall Compilation
#### Public Compile
##### HDMI

```
./FFTools/make.sh -j24 -d rk3399-firefly-aio -l rk3399_firefly_aio-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aio-userdebug
```

#### Display DM-M10R800 Compile
##### LVDS+HDMI

```
./FFTools/make.sh  -d rk3399-firefly-aio-lvds-HSX101H40C -j8 -l rk3399_firefly_aio_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aio_lvds-userdebug
```

#### Dual Camera SV-TAYSH-TQ Compile
##### HDMI+SV-TAYSH-TQ

<font color=#ff0000 size=4>**note:**</font> <br />
<font color=#ff0000>The camera only supports aio-3399j Standard Edition and does not support HDMI_In version</font><br />
* Modify `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aio.dtsi`
```
    xc7160b@1b{
+    status = "disabled";
    };
    xc7160f@1b{
+    status = "disabled";
    };

    XC6130b@23{
+    status = "okay";
    };
    XC7022b@1b{
+    status = "okay";
    };
```

* Compile
```
./FFTools/make.sh -j24 -d rk3399-firefly-aio -l rk3399_firefly_aio-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aio-userdebug
```


### Compile step by step AIO-3399Pro-JD4

Before compiling, execute the following command to configure environment variables:

```
source ./FFTools/build.sh
```

* Compile kernel:

```
cd ~/proj/rk3399_Android10.0/kernel/
make ARCH=arm64 firefly_defconfig android-10.config rk3399.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_firefly_aio/boot.img rk3399-firefly-aio.img -j8
```

* Note：If you going to do kernel debug，you need pack the resource.img and kernel.img together into boot.img,then  upgrade the boot,img into boot partition.

* Compile uboot：

```
cd ~/proj/rk3399_Android10.0/u-boot/
./make.sh rk3399
```

* Compile Android：

```
cd ~/proj/rk3399_Android10.0/
lunch rk3399_firefly_aio-userdebug
make -j8
./mkimage.sh
```

## Package into a unified firmware 

After compiling, execute the following command first, then use Firefly official script to package into a unified SD card firmware, execute the following command:

```
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_aio-userdebug
```

According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

Packing the unified firmware `update.img` under Windows is also very simple. Copy the compiled files to the `rockdev/Image` directory of AndroidTool, and then run the `mkupdate.bat` batch file under the `rockdev` directory. Create `update.img` and store it in the `rockdev/Image` directory.

## Some Introduction about Compiling 
###  Android 10.0 can't be written directly kernel.img and resource.img 
Android 10.0 kernel.img and resource.img Included in boot.img After the kernel is updated and compiled, it needs to be executed in the Android root directory mkimage.sh Repackage boot.img .After packing, download boot.img  in rockdev directory .You can compile the kernel separately by using the following instruction.
###  Compiling kernel generation separately boot.img 
Principle of compilation: in the kernel directory, the generated kernel.img And resource.img Replace with old boot.img So you need to use boot when compiling  IMG = XXX parameter specification boot.img The command is as follows:
``` shell
cd kernel
make ARCH=arm64 firefly_defconfig android-10.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_firefly_aio/boot.img rk3399-firefly-aio.img -j24
```
After compiling, you can directly download the boot.img To the machine.

## Partition image
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

## Flash Image 

Reference: [《Flash Image》](03-upgrade_firmware.md)