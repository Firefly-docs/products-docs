# 编译主核心模组固件
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

### 配置
#### Core-3588JD4

```
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_debian_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 `AIBOX-PRO-G2-3588JD4_Debian.XXX.img` 

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

### 配置
### RK3588

```
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_ubuntu_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 ``AIBOX-PRO-G2-3588JD4_Ubuntu.XXX.img`

## 导出主核心模组的 rootfs
参考 [导出设备系统](/docs/tools/development-tool/Rootfs-Export-Tool/ff-export-rootfs)




