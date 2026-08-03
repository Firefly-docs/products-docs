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

!INCLUDE "./linux6.1_compile_debian_aibox-pro-3588.md"

## 编译 Ubuntu 固件
<font color=red> 先获取 SDK。 </font>

!INCLUDE "./linux6.1_compile_ubuntu_aibox-pro-3588.md"

## 导出主核心模组的 rootfs
参考 [导出设备系统](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/first_use.html#dao-chu-she-bei-xi-tong)

