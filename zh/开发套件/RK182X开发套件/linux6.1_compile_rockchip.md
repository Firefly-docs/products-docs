# 主核心模组

## 获取 SDK

根据主核心模组，联系销售（`sales@t-firefly.com`）获取 **RK3588 Kernel 6.1 SDK** 或 **RK3576 Kernel 6.1 SDK**，并阅读 SDK 中的 `README` 文档。

<font color=red>

**注意：**
<br>
**1.** SDK 采用交叉编译，请在 `x86_64` 电脑上使用 SDK，不要将 SDK 下载到开发套件上。<br>
**2.** 建议使用 Ubuntu 20.04（实体机或 Docker 容器）进行编译，其他操作系统可能导致编译失败。<br>
**3.** 不要在虚拟机共享目录或包含非英文字符的目录中存放、解压 SDK。<br>
**4.** 获取和编译 SDK 请使用普通用户，只有安装系统软件包时才需要 root 权限。

</font>

* RK3588 SDK：至少更新到 `rk3588/linux6.1_release_v1.3.0e`。
* RK3576 SDK：至少更新到 `rk3576/linux_release_v1.3.0a`。

开发套件支持以下主核心模组的 Debian 12 和 Ubuntu 固件：

* Core-3588JD4
* Core-3588SJD4 AI
* Core-3576JD4

请选择需要编译的操作系统：

* [编译 Debian 固件](linux6.1_compile_debian.md)
* [编译 Ubuntu 固件](linux6.1_compile_ubuntu.md)

## 导出主核心模组 rootfs

rootfs 导出方法请参考[导出设备系统](/docs/tools/development-tool/Rootfs-Export-Tool/ff-export-rootfs)。
