# Compile Android9.0

### Download Android SDK

Since the Android SDK source code package is relatively large, you can obtain the Android 9.0 source code package in the following ways:
[Download link](http://en.t-firefly.com/doc/download/69.html#other_247)

After the download is complete, verify the MD5 code before decompression：
```
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.001
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.002

6e1d3969c8a0f643522727ff07800bb5  rk3399pro_firefly_android9.0_20191126.7z.001
a6b8d6a775c3d5ed28f4d41cb210a84d  rk3399pro_firefly_android9.0_20191126.7z.002
```

Then unzip：
```
cd ~/proj/
7z x ./rk3399pro_firefly_android9.0_20191126.7z.001 -oAIO-3399Pro
cd ./AIO-3399Pro
git reset --hard
```



The following is how to update from gitlab：
```
#1. Enter the SDK root directory
cd ~/proj/AIO-3399Pro

#2. Download the remote bundle warehouse
git clone https://gitlab.com/TeeFirefly/rk3399pro-pie-bundle.git .bundle

#3. If downloading the warehouse fails, the current bundle warehouse occupies a large space, so there may be stuck or failed during synchronization.
# You can download it from the Baidu Cloud link below and unzip it to the SDK root directory. The unzip command is as follows:

7z x rk3399pro-pie-bundle.7z  -r -o. && mv rk3399pro-pie-bundle/ .bundle/

#4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

#5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD
```

You can also view the source code online at the following address:
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)

## AIO-3399ProC product compilation method

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
[EN_DOW_LINK]: http://en.t-firefly.com/doc/download/69.html

## Partition mirroring

* boot.img: Android initramfs image, including the basic file system of the Android root directory, which is responsible for initializing and loading the system partition.
* system.img: Android file system partition image in ext4 file system format.
* kernel.img: Kernel image.
* resource.img: Resource image, including boot image and kernel device tree.
* misc.img: misc partition image, responsible for the switch of boot mode and the transfer of parameters in emergency mode.
* recovery.img: Recovery mode image.
* rk3399_loader_v1.12.112.bin: Loader file.
* uboot.img: U-Boot image file.
* trust.img: Arm trusted file (ATF) image file.
* parameter.txt: partition layout and kernel command line.
* vendor.img: TODO
* oem.img: TODO
* baseparameter.img: TODO


## Other Android versions
* <font color=#ff0000 size=3>Main maintenance:</font>

   ["Compile Android9.0"](compile_android9.0_firmware.md)  
* Support but not maintain:


