### 整体编译
#### 公版编译
* HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4 -j8 -l rk3399pro_firefly_aiojd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4-userdebug
```

#### 显示屏 DM-M10R800 编译
* LVDS + HDMI
```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4-lvds-HSX101H40C -j8 -l rk3399pro_firefly_aiojd4_lvds-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4_lvds-userdebug
```

#### 显示屏 DM-M10R800 V2 MIPI屏模组 编译
* MIPI + HDMI
```
./FFTools/make.sh -j8 -d rk3399pro-firefly-aiojd4-mipi_M101014_BE45_A1 -l rk3399pro_firefly_aiojd4_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4_mipi-userdebug
```

#### 双目摄像头 SV-TAYSH-TQ 编译
* HDMI + SV-TAYSH-TQ

修改kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4.dtsi
```
        xc7160b: xc7160b@1b {
+                status = "disabled";
                reset-gpios = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
        };

        xc7160f: xc7160f@1b {
+                status = "disabled";
                reset-gpios = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
        };

        XC7022b: XC7022b@1b{
+                status = "okay";
                reset-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
        };

        XC6130b: XC6130b@23{
+                status = "okay";
                reset-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
        };
```
* 编译
```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4 -j8 -l rk3399pro_firefly_aiojd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4-userdebug
```

### 手动编译 AIO-3399C Android 9.0


编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/AIO-3399C/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3399pro_firefly_aiojd4/boot.img rk3399pro-firefly-aiojd4.img
```

* 注意：若进行内核 debug，需要将 resource.img 和 kernel.img 打包进去 boot.img 后对 boot 分区进行烧写才能生效。

* 编译 uboot：

```
cd ~/proj/AIO-3399C/u-boot/
./make.sh rk3399pro
```

* 编译 Android：

```
cd ~/proj/AIO-3399C/
source build/envsetup.sh
lunch rk3399pro_firefly_aiojd4-userdebug
make -j8
./mkimage.sh
```