# Linux 开发

## 编译主核心模组固件
## 获取 SDK

根据**主核心模组**，联系销售 (sales@t-firefly.com) 获取 **RK3588 Kernel6.1 SDK**下载链接, 并且阅读下载链接的 **readme** 文档。

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

## 编译 Debian 固件
<font color=red> 先获取 SDK。 </font>

### rootfs
* 下载根文件系统：[Debian 根文件系统(64位) Kernel6.1](https://community.t-firefly.com/doc/download/414)，请使用网盘中 kernel-6.1 目录下的文件系统。
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

### 配置
#### Core-3588JD4

```
./build.sh firefly_rk3588_aibox-pro-3588jd4_rk182x_debian_defconfig
```

#### Core-3576JD4

```
./build.sh firefly_rk3576_aibox-pro-3576jd4-rk182x-m2_debian_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 `AIBOX-PRO-3588JD4_Debian.XXX.img` 
## 编译 Ubuntu 固件
<font color=red> 先获取 SDK。 </font>

### rootfs
* 下载根文件系统：[Ubuntu 根文件系统(64位) Kernel6.1](https://community.t-firefly.com/doc/download/414)，请使用网盘中 kernel-6.1 目录下的文件系统。
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

### 配置
### RK3588

```
./build.sh firefly_rk3588_aibox-pro-3588jd4_rk182x_ubuntu_defconfig
```

### RK3576

```
./build.sh firefly_rk3576_aibox-pro-3576jd4-rk182x-m2_ubuntu_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 ``AIBOX-PRO-3588JD4_Ubuntu.XXX.img`
## 导出主核心模组的 rootfs
参考 [导出设备系统](/docs/tools/development-tool/Rootfs-Export-Tool/ff-export-rootfs)



## 编译 RK1820/RK1828 安装包

## 获取 SDK

请联系销售 (sales@t-firefly.com) 获取 **RK182X SDK** 下载链接。

<font color=red>

**注意：**
<br>
**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**<br>
**2. 编译环境请使用 Ubuntu20.04或Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**<br>
**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**<br>
**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</font>

<br>
比如，SDK 压缩包是 `RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.X.X.tgz`。（具体版本以网盘名称最新发布为准）

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.X.X.tgz
.repo/repo/repo sync -l
```

### Bundle 更新
1.1.0a 后续版本将以 bundle 的形式更新，以减少下载时间。
下载并解压上述 SDK 基础包、完成同步后，将 bundle 包放置在 SDK 根目录下：

```
rk182x_sdk/
├── .repo/
└── bundle_xx_to_xx.tgz
```

解压对应的 bundle 包：

```
tar xzf bundle_xx_to_xx.tgz
```

在 SDK 根目录运行 bundle 内的脚本：
```
./bundle_xx_to_xx/bundle_update.sh
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

选择 `4`


```
Select Security Boot Mode:
1) Disable Secure Boot
2) Enable Secure Boot
3) Cancel
#? 
```

选择 `1`
> 该选项用于对固件进行相关安全加密，误操作可能导致固件后续无法升级，请谨慎选择。相关资料请参考1828SDK_Path/docs/Develop/Rockchip_RK1820_RK1828_User_Guide_SecureBoot_CN.pdf

## 编译
```
./build.sh
```

生成的软件安装包在 `output/firmware/rknn3_rk182x_m2_installer_arm64.tgz`

## 安装
手动安装 RK1820/RK1828 软件包，按如下步骤操作：
* 拷贝 `rknn3_rk182x_m2_installer_arm64.tgz` 到主控端
* 解压 `tar xzf rknn3_rk182x_m2_installer_arm64.tgz`
* 安装 `./install.sh`
    * 安装重启后， RK3588 或者 RK3576 端系统会在启动后， ⾃动下载 RK182X 的固件，并启动后台服务程序。


## 其他
### 版本 V 1.1.0
```
sudo rknn-smi -v
rknn-smi version              : 1.3.0
PCIe driver version           : 3.3.1
RC chips connect version      : 3.3.2
EP chips connect version      : 0.0.2
PCIe Device 0 firmware version: 1.1.0
rknn3 API version             : 1.1.0

```

## FAQ

### 当前支持的模型

当前支持的模型和详细说明，请参考 SDK 中的 `/home/zhang/rk182x_self/rknn/rknn3-runtime/doc/CN/00_RKNN3_SDK_发布说明_V1.1.0.pdf`。

### rknn3 API Version 显示 NA
安装一下binutils, 部分rootfs可能没有strings指令

```sh
sudo apt update
sudo apt install binutils
```

