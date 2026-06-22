### 整体编译

<font color=#FF0000>注意：由于占用摄像头接口因此编译前需先注释摄像头配置，打开扩展板配置</font>
```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
index 6d376df7e6e..289a8347c07 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dts
@@ -7,9 +7,9 @@
 #include "roc-rk3588s-pc.dtsi"
-//#include "roc-rk3588s-pc-ext.dtsi"
+#include "roc-rk3588s-pc-ext.dtsi"

-#include "rk3588-roc-pc-cam-8ms1m.dtsi"
+//#include "rk3588-roc-pc-cam-8ms1m.dtsi"
```

#### HDMI 固件编译 
```
./FFTools/make.sh -d  -j8 -l rk3576_firefly_ext_icore_3576q38-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```

### 分步编译

* 编译 kernel：

```
cd ~/path/to/sdk/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 
msk ARCH=arm64   BOOT_IMG=../rockdev/rk3576_firefly_ext_icore_3576q38/boot.img .img -j8

```

* 编译 uboot：

```
cd ~/path/to/sdk/u-boot/
./make.sh rk3576
```

* 编译 Android：

```
cd ~/path/to/sdk/
source build/envsetup.sh
lunch rk3576_firefly_ext_icore_3576q38-userdebug
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_ext_icore_3576q38-userdebug
```
打包完成后将在rockdev/rk3576_firefly_ext_icore_3576q38/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。