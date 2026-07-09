# 编译主核心模组固件
## 获取 SDK

根据**主核心模组**，联系销售 (sales@t-firefly.com) 获取 **RK3588 Kernel6.1 SDK** 或者 **RK3576 Kernel6.1 SDK** 下载链接, 并且阅读下载链接的 **readme** 文档。

<font color=red>

**注意：**
<br>
**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**<br>
**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**<br>
**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**<br>
**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</font>

* RK3588 SDK
    * 至少更新到 `rk3588/linux6.1_release_v1.3.0e`
* RK3576 SDK
    * 至少更新到 `rk3576/linux_release_v1.3.0a`

## 编译 Debian 固件
<font color=red> 先获取 SDK。 </font>

### rootfs
* 下载根文件系统：[Debian 根文件系统(64位) Kernel6.1](https://www.t-firefly.com/doc/download/290.html)，请使用网盘中 kernel-6.1 目录下的文件系统。
* 解压 rootfs 并链接 rootfs

#### RK3588 
```
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

#### RK3576 
```
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3576_debian_rootfs.img
cd ..
```

### 配置
#### RK3588
##### Core-3588JD4

```
./build.sh firefly_rk3588_aio-gs1n2-3588jd4-rk182x_debian_defconfig
```

##### Core-3588SJD4 AI

```
./build.sh firefly_rk3588_aio-gs1n2-3588sjd4-ai-rk182x_debian_defconfig
```

#### RK3576
##### Core-3576JD4

```
./build.sh firefly_rk3576_aio-gs1n2-3576jd4-rk182x_debian_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 `AIO-GS1N2-3588JD4-RK182X_Debian.XXX.img` 

## 编译 Ubuntu 固件
<font color=red> 先获取 SDK。 </font>

### rootfs
* 下载根文件系统：[Ubuntu 根文件系统(64位) Kernel6.1](https://www.t-firefly.com/doc/download/290.html)，请使用网盘中 kernel-6.1 目录下的文件系统。
* 解压 rootfs 并链接 rootfs

#### RK3588 
```
# 解压
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

#### RK3576 
```
# 解压
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3576_ubuntu_rootfs.img
cd ..
```

### 配置
#### RK3588
##### Core-3588JD4

```
./build.sh firefly_rk3588_aio-gs1n2-3588jd4-rk182x_ubuntu_defconfig
```

##### Core-3588SJD4 AI

```
./build.sh firefly_rk3588_aio-gs1n2-3588sjd4-ai-rk182x_ubuntu_defconfig
```

#### RK3576
##### Core-3576JD4

```
./build.sh firefly_rk3576_aio-gs1n2-3576jd4-rk182x_ubuntu_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 `AIO-GS1N2-3588JD4-RK182X_Ubuntu.XXX.img`

## 导出主核心模组的 rootfs
参考 [导出设备系统](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/first_use.html#dao-chu-she-bei-xi-tong)

# 编译 RK1820/RK1828 安装包
## 获取 SDK

请联系销售 (sales@t-firefly.com) 获取 **RK182X SDK** 下载链接。

<font color=red>

**注意：**
<br>
**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**<br>
**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**<br>
**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**<br>
**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</font>

<br>
比如，SDK 压缩包是 `rk182x_linux_release_20260611_v1.0.4a.tgz`。

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf rk182x_linux_release_20260611_v1.0.4a.tgz
.repo/repo/repo sync -l
```

## 配置 
通过 `./build.sh config` 配置。

```
Select board type:
1) RK182X EVB1
2) RK182X SODIMM
3) RK182X SODIMM USB
4) RK182X M2
5) Cancel
#? 
```

选择 `2`

## 编译
```
./build.sh
```

生成的软件安装包在 `output/firmware/rknn3_rk182x_sodimm_installer_arm64.tgz`

## 安装
手动安装 RK1820/RK1828 软件包，按如下步骤操作：
* 拷贝 `rknn3_rk182x_sodimm_installer_arm64.tgz` 到 RK3588 或者 RK3576 端
* 解压 `tar xzf rknn3_rk182x_sodimm_installer_arm64.tgz`
* 安装 `./install.sh`
    * 安装重启后， RK3588 或者 RK3576 端系统会在启动后， ⾃动下载 RK182X 的固件，并启动后台服务程序。


## 其他
### 版本 V1.0.0
```
sudo rknn-smi -v
rknn-smi version : 1.1.0
PCIe driver version : 3.3.0
RC chips connect version : 3.2.0
EP chips connect version : 3.2.0
PCIe Device 0 firmware version: 1.0.0
rknn3 API version : 1.0.0
```

