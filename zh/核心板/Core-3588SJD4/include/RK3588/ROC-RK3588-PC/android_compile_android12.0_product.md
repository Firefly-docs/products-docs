### 整体编译

#### HDMI 固件编译 
```
./FFTools/make.sh -d aio-3588sjd4 -j8 -l aio_3588sjd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```  

### 分步编译

* 编译 kernel：

```
cd ~/proj/RK3588_Android12.0/kernel-5.10
export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
msk ARCH=arm64   BOOT_IMG=../rockdev/Image-aio_3588sjd4/boot.img aio-3588sjd4.img -j8

```

* 编译 uboot：

```
cd ~/proj/RK3588_Android12.0/u-boot/
./make.sh rk3588
```

* 编译 Android：

```
cd ~/proj/RK3588_Android12.0/
source build/envsetup.sh
lunch aio_3588sjd4-userdebug
make installclean
make -j8
./mkimage.sh
```

### 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l aio_3588sjd4-userdebug
```
打包完成后将在rockdev/Image-aio_3588sjd4/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。