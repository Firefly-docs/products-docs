### 整体编译
#### 公版编译
* HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aioc -j8 -l rk3399pro_firefly_aioc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc-userdebug
```

#### 显示屏 DM-M10R800 编译
* LVDS + HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aioc-lvds-HSX101H40C -j8 -l rk3399pro_firefly_aioc_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc_lvds-userdebug
```

#### 显示屏 DM-M10R800 V2 MIPI屏模组 编译
* MIPI + HDMI
```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aioc-mipi_M101014_BE45_A1 -l rk3399pro_firefly_aioc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc_mipi-userdebug
```

#### 双目摄像头 SV-TAYSH-TQ 编译
* HDMI + SV-TAYSH-TQ

修改kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aioc.dtsi
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
* 编译
```
./FFTools/make.sh  -d rk3399pro-firefly-aioc -j8 -l rk3399pro_firefly_aioc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aioc-userdebug
```

### 手动编译 EC-R3399PC Android 9.0

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/AIO-3399ProC/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399pro_firefly_aioc/boot.img rk3399pro-firefly-aioc.img
```

* 注意：若进行内核 debug，需要将 resource.img 和 kernel.img 打包进去 boot.img 后对 boot 分区进行烧写才能生效。

* 编译 uboot：

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