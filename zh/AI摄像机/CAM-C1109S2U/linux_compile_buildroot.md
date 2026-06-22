## 编译 Buildroot 固件

本章介绍 Buildroot 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

<font color=red>**本教程的编译部分适用于 v2.2.5f 以上 SDK 版本**</font>

```
$ realpath .repo/manifest.xml
/home/lvsx/project/rv1126_rv1109/.repo/manifests/rv1126_rv1109/rv1126_rv1109_linux_release_20240117_v2.2.5f.xml
```
### 准备工作

#### 搭建编译环境

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip \
device-tree-compiler ncurses-dev \
```

### 编译 SDK

#### 编译前配置

在 `device/rockchip/rv1126_rv1109/` 目录下，有不同板型的配置文件。

`rv1126_rv1109_linux_release` SDK 和 `rv1126_rv1109_linux_bsp_release` SDK 选择配置文件：
```bash



# V2 MIPI 屏幕（老版本屏幕，该屏幕已停产）
./build.sh cam-crv1109s2u-facial_gate-BE-45.mk 

# V3 MIPI 屏幕
./build.sh cam-crv1109s2u-facial_gate-mipi101-BSD1218-A101KL68.mk


```

`rv1126_rv1109_linux_ai_camera_release` SDK 和 `rv1126_rv1109_linux_ai_camera_bsp_release` SDK 选择配置文件：


由于 uvc 固件一般不需要使用 mipi 屏幕来显示。如果有 mipi 显示需求可以选择对应的屏幕进行固件编译。

```bash
# V2 MIPI 屏幕（老版本屏幕，该屏幕已停产）
./build.sh cam-crv1109s2u-uvcc-BE-45.mk 

# V3 MIPI 屏幕
./build.sh cam-crv1109s2u-uvcc-mipi101-BSD1218-A101KL68.mk 
```

#### 编译

##### 全自动编译

全自动编译会执行上述编译、打包操作，生成 RK 固件。

```bash
./build.sh
```

打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

##### 部分编译

* 编译 u-boot

```bash
./build.sh uboot
```

* 编译 kernel

```bash
./build.sh kernel
```

* 编译 recovery

```bash
./build.sh recovery
```

* 编译 buildroot

```bash
./build.sh rootfs
```

* 更新各部分镜像链接到 `rockdev/` 目录：

```bash
./mkfirmware.sh
```

* 打包固件，生成的完整固件会保存到 `rockdev/pack/` 目录。

```bash
./build.sh updateimg
```