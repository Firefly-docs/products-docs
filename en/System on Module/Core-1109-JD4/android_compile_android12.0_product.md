### 整体编译

#### HDMI 固件编译 
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```
#### 显示屏 DM-M10R800 V2 固件编译：
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### 双目摄像头 CAM-2MS2MF 编译：
##### HDMI+CAM-2MS2MF
* 修改 dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
@@ -8,8 +8,8 @@
 
 #include "rk3588-firefly-itx-3588j.dtsi"
-#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
-//#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
+//#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
+#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```

* 编译
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

#### HDMI TO MIPI_CSI(RK628D) 编译
* 修改 dts
```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dts
@@ -7,9 +7,9 @@
 /dts-v1/;
 
 #include "rk3588-firefly-itx-3588j.dtsi"
-#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
+//#include "rk3588-firefly-itx-cam-8ms1m.dtsi"
 //#include "rk3588-firefly-itx-cam-2ms2mf.dtsi"
-//#include "rk3588-firefly-itx-3588j-tf-hdmi-mipi-rk628.dtsi"
+#include "rk3588-firefly-itx-3588j-tf-hdmi-mipi-rk628.dtsi"
 #include "rk3588-firefly-demo.dtsi"
```
* 编译
```
./FFTools/make.sh -d  -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```

### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 
msk ARCH=arm64   BOOT_IMG=../rockdev//boot.img .img -j8

```

* 编译 uboot：

```
cd ~/proj/RK3588_Android12.0/u-boot/
./make.sh rv1109
```

* 编译 Android：

```
cd ~/proj/RK3588_Android12.0/
source build/envsetup.sh
lunch 
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l 
```
打包完成后将在rockdev// 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。