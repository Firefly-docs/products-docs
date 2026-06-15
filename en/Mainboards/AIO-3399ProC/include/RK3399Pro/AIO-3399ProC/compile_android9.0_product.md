### Overall Compilation
#### Public Compile
##### HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aioc -j8 -l rk3399pro_firefly_aioc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc-userdebug
```

#### Display DM-M10R800 Compile
##### LVDS + HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aioc-lvds-HSX101H40C -j8 -l rk3399pro_firefly_aioc_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc_lvds-userdebug
```

#### Display DM-M10R800 V2 MIPI Compile
##### LVDS + HDMI
```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aioc-mipi_M101014_BE45_A1 -l rk3399pro_firefly_aioc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc_mipi-userdebug
```

#### Dual Camera SV-TAYSH-TQ Compile
* HDMI + SV-TAYSH-TQ

kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aioc.dtsi
```
        xc7160b: xc7160b@1b {
+                status = "disabled";
        };

        xc7160f: xc7160f@1b {
+                status = "disabled";
        };

        XC7022b: XC7022b@1b{
+                status = "okay";
        };

        XC6130b: XC6130b@23{
+                status = "okay";
        };
```
* Complie
```
./FFTools/make.sh  -d rk3399pro-firefly-aioc -j8 -l rk3399pro_firefly_aioc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc-userdebug
```

### Manually compile AIO-3399ProC Android 9.0

Before compiling, execute the following command to configure environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:

```
cd ~/proj/AIO-3399ProC/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399pro_firefly_aioc/boot.img rk3399pro-firefly-aioc.img
```

* Note: If you are performing kernel debugging, you need to package resource.img and kernel.img into boot.img and then burn the boot partition to take effect. Note: If you are performing kernel debugging, you need to package resource.img and kernel.img into boot.img and then burn the boot partition to take effect.

* Compile uboot:

```
cd ~/proj/AIO-3399ProC/u-boot/
./make.sh rk3399pro
```

* 编译 Android：

```
cd ~/proj/AIO-3399ProC/
source build/envsetup.sh
lunch rk3399pro_firefly_aioc-userdebug
make -j8
./mkimage.sh
```