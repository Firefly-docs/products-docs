### Overall Compilation

**Note:** Since ROC-RK3399-PC-PLUS is the Hardware iteration version of [ROC-RK3399-PC-PLUS](https://wiki.t-firefly.com/en/ROC-RK3399-PC-PLUS/started.html), so the compilation method of software is consistent. Finally generated firmware, such as `ROC-RK3399-PC-Pro_ xxx.img` is compatible with [ROC-RK3399-PC-PLUS](https://wiki.t-firefly.com/en/ROC-RK3399-PC-PLUS/started.html).

#### Public Compile
##### HDMI+DP
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

#### Display DM-M10R800 V2 Compile
##### MIPI_DSI0+HDMI
```
./FFTools/make.sh -j8 -d rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1 -l rk3399_roc_pc_plus_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus_mipi-userdebug
```

#### Camera OV13850 Compile
##### HDMI+OV13850

* Modify `kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi`
```
 &i2c1{

-    XC6130b@23{
+    ov13850b@10{
     status = "okay";
     };
-    XC7022b@1b{
+    ov13850f@10{
     status = "okay";
     };

-    /delete-node/ ov13850b@10;
-    /delete-node/ ov13850f@10;
+    /delete-node/ XC6130b@23;
+    /delete-node/ XC7022b@1b;

 };
```

* Compile
```
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```

#### Dual Camera SV-TAYSH-TQ Compile
##### HDMI+SV-TAYSH-TQ

* Modify `kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dtsi`
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
./FFTools/make.sh -d rk3399-roc-pc-plus -j8 -l rk3399_roc_pc_plus-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
```


### Step by Step Compilation 

Take compiling HDMI firmware as an example,Before compiling, execute the following command to configure environment variables:

```
source ./FFTools/build.sh
```

* Compile kernel:

```
cd ~/proj/rk3399_Android10.0/kernel/
make ARCH=arm64 firefly_defconfig android-10.config rk3399.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399_roc_pc_plus/boot.img rk3399-roc-pc-plus.img -j8
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
lunch rk3399_roc_pc_plus-userdebug
make -j8
./mkimage.sh
```

* Packaged into unified firmware 

	After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
	```
	./FFTools/mkupdate/mkupdate.sh -l rk3399_roc_pc_plus-userdebug
	```
	According to different -l XXX-userdebug parameters, the packaged unified firmware will be stored in different directories (rockdev/image-XXX/): product name XXX_XXX_date XXX.img

	It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create update.img and store it in rockdev \ Image directory.

### Partition image
* boot.img : include ramdis、kernel、dtb
* dtbo.img : Device Tree Overlays
* kernel.img : Currently, it cannot be burned separately, it needs to be packaged into boot.img to burn and write
* MiniLoaderAll.bin : include first loader
* misc.img : includerecovery-wipe boot flag information ，enter recovery afte v 
* odm.img : include android odm，included in super.img partition ,upgrade alone by the fastboot tool 
* parameter.txt : include partition information
* pcba_small_misc.img : include pcba boot flag information，enter the simple pcba mode after upgrade 
* pcba_whole_misc.img : include pcba boot flag information，enter the full pcba mode after upgrade
* recovery.img : include recovery-ramdis、kernel、dtb
* resource.img : include dtb，kernel and  uboot phases log and the uboot charging  logo,it should be packed into boot.img,then upgrade it
* super.img : include odm、vendor、system partition content
* system.img : include android system，included in super.img partition ,upgrade alone by the fastboot tool 
* trust.img : include BL31、BL32
* uboot.img : include uboot.img
* vbmeta.img : include avb，for AVB verify
* vendor.img : include android vendor，included in super.img partition ,upgrade alone by the fastboot tool 
* update.img : include all the img file above，use tools to upgrade the whole firmware package 

## Flash Image 

Reference: [《Flash Image》](03-upgrade_firmware.md)