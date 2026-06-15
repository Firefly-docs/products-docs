### 公版编译
#### HDMI
```
./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
```

### 显示屏 DM-M10R800 V2 编译
#### MIPI_DSI1+HDMI
```
./FFTools/make.sh -d rk3568-firefly-itx-3568q-mipi101_M101014_BE45_A1 -j8 -l rk3568_firefly_itx_3568q-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_itx_3568q-userdebug
```

### 显示屏 DM-M10R800 V3S 编译
#### MIPI_DSI1+HDMI
```
./FFTools/make.sh -d rk3568-firefly-itx-3568q-mipi101_BSD1218_A101KL68 -j8 -l rk3568_firefly_itx_3568q-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_itx_3568q-userdebug
```

### 双目摄像头 CAM-2MS2MF 编译
#### HDMI+CAM-2MS2MF
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts
    index d784287..fbe7b6b 100755
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-itx-3568q.dts
    @@ -21,8 +21,8 @@
      * using dual camera gc2053/gc2093   ----> rk3568-firefly-itx-3568q-cam-2ms2m.dtsi
      * using hdmi-in module rk628d   ----> rk3568-firefly-itx-3568q-tf-hdmi-mipi-rk628.dtsi
      */
    -#include "rk3568-firefly-itx-3568q-cam-8ms1m.dtsi"
    -//#include "rk3568-firefly-itx-3568q-cam-2ms2m.dtsi"
    +//#include "rk3568-firefly-itx-3568q-cam-8ms1m.dtsi"
    +#include "rk3568-firefly-itx-3568q-cam-2ms2m.dtsi"
     //#include "rk3568-firefly-itx-3568q-tf-hdmi-mipi-rk628.dtsi"
    ```

* 编译

    ```
    ./FFTools/make.sh -d rk3568-firefly-aioj -j8 -l rk3568_firefly_aioj-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
    ```

## 手动编译 Core-3568J Android 11.0

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 firefly_defconfig android-11.config rk356x.config firefly_wifi.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3568_firefly_aioj/boot.img rk3568-firefly-aioj.img -j8
```

**<font color=#ff0000 size=3>注意：进行内核debug的时候，单编译生成boot.img如果出现运行错误，可参考[FAQs](faqs.html#android-sdk-dan-du-bian-yi-sheng-cheng-de-boot-img-yu-dao-wen-ti)</font>**

* 编译 uboot：

```
cd ~/proj/RK356X/u-boot/
./make.sh rk3568
```

* 编译 Android：

```
cd ~/proj/RK356X/
source build/envsetup.sh
lunch rk3568_firefly_aioj-userdebug
make installclean
make -j8
./mkimage.sh
```

## 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj-userdebug
```
打包完成后将在rockdev/Image-XXX/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。