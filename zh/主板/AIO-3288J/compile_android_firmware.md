# 编译 Android 5.1 固件

## 下载 Android SDK

**Android SDK 源码包比较大(约6.3G),可以通过如下方式获取源码包：**

* [[下载链接]](http://www.t-firefly.com/doc/download/16.html#other_35)

下载完成后先验证一下 MD5 码：

```bash
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
$md5sum ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.002

3d7a59a84e059cd55a68304d47a5462b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.001
1979bd263b51bec27ec10eceb500731b  firefly_rk3288_rk3128_android5.1_git_20211216.7z.002
```

确认无误后，就可以解压：

```bash
mkdir -p ~/proj/firefly_rk3288_android5.1
cd ~/proj/firefly_rk3288_android5.1
7z x ~/firefly_rk3288_rk3128_android5.1_git_20211216.7z.001 -r -o.
git reset --hard
```

以后就可以直接从 gitlab 处更新

```bash
git pull gitlab Firefly-RK3288:Firefly-RK3288
```

## 编译方法

### 整体编译
#### 公版编译
##### HDMI


```bash
./FFTools/make.sh -d firefly-rk3288-aio-3288j -j8 -l rk3288_aio_3288j_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3288_aio_3288j_box-userdebug
```



#### 显示屏 DM-M10R800 编译
##### LVDS+HDMI

```
./FFTools/make.sh -d firefly-rk3288-aio-3288j_lvds -j8 -l rk3288_aio_3288j_box-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3288_aio_3288j_box-userdebug
```


### 分步编译

* 编译 uboot

```bash
cd ~/proj/firefly_rk3288_android5.1/u-boot
make rk3288_defconfig
make -j8
```

* 编译 kernel

```bash
cd ~/proj/firefly_rk3288_android5.1/kernel
make firefly_defconfig
make -j8 firefly-rk3288-aio-3288j.img
```


* 编译 Android

```bash
cd ~/proj/firefly_rk3288_android5.1/
source build.sh
lunch rk3288_aio_3288j_box-userdebug
make -j8
./mkimage.sh
```

### 构建变体说明  

默认的目标构建变体(TARGET_BUILD_VARIANT)为 userdebug。常用变体有三种，分别是用户(user)、用户调试(userdebug)和工程模式(eng)，其区别如下：

* user
    + 仅安装标签为 user 的模块
    + 设定属性 ro.secure=1，打开安全检查功能
    + 设定属性 ro.debuggable=0，关闭应用调试功能
    + 默认关闭 adb 功能
    + 打开 Proguard 混淆器
    + 打开 DEXPREOPT 预先编译优化

* userdebug
    + 安装标签为 user、debug 的模块
    + 设定属性 ro.secure=1，打开安全检查功能
    + 设定属性 ro.debuggable=1，启用应用调试功能
    + 默认打开 adb 功能
    + 打开 Proguard 混淆器
    + 打开 DEXPREOPT 预先编译优化

* eng
    + 安装标签为 user、debug、eng 的模块
    + 设定属性 ro.secure=0，关闭安全检查功能
    + 设定属性 ro.debuggable=1，启用应用调试功能
    + 设定属性 ro.kernel.android.checkjni=1，启用 JNI 调用检查
    + 默认打开 adb 功能
    + 关闭 Proguard 混淆器
    + 关闭 DEXPREOPT 预先编译优化

如果目标构建变体为 user，则 adb 无法获取 root 权限。
要选择目标构建变体，可以在 make 命令行加入参数，例如：

```bash
make -j8 PRODUCT-rk3288_aio_3288j_box-user
make -j8 PRODUCT-rk3288_aio_3288j_box-userdebug
make -j8 PRODUCT-rk3288_aio_3288j_box-eng
```

## 烧写分区映像

上一步骤的 `./mkimage.sh` 会重新打包 boot.img 和 system.img, 并将其它相关的映像文件拷贝到目录 `rockdev/Image-rk3288_aio_3288j_box/` 中。

* 以下列出一般固件用到的映像文件：
    + boot.img ：Android 的初始文件映像，负责初始化并加载 system 分区。
    + kernel.img ：内核映像。
    + misc.img ：misc 分区映像，负责启动模式切换和急救模式的参数传递。
    + recovery.img ：急救模式映像。
    + resource.img ：资源映像，内含开机图片和内核的设备树信息。
    + system.img ：Android 的 system 分区映像，ext4 文件系统格式。

请参照[升级固件](upgrade_firmware.md) 一文来烧写分区映像文件。
如果使用的是 Windows 系统，将上述映像文件拷贝到 AndroidTool （Windows 下的固件升级工具）的 `rockdev\Image` 目录中，之后参照升级文档烧写分区映像即可，这样的好处是使用默认配置即可，不用修改文件的路径。

## 打包成统一固件

编译完可以使用 Firefly 的脚本打包成统一固件，执行如下命令：

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3288_aio_3288j_box-userdebug
```

打包完成后固件会生成在 `rockdev/Image-rk3288_aio_3288j_box/` 目录

在 Windows 下打包统一固件 update.img 很简单，按上一步骤将文件拷贝到 AndroidTool 的 `rockdev\Image` 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 `rockdev\Image` 目录里。

update.img 方便固件的发布，供终端用户升级系统使用。一般开发时使用分区映像比较方便。