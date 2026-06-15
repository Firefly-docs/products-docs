# Firefly Ubuntu Desktop 介绍

IPC-M10R800-A3568J Ubuntu Desktop 系统基于 Ubuntu 64bit 系统构建，目前主要有 Ubuntu20.04 版本。具有以下特性：

1.  桌面环境采用 LXQT 或 XFCE4，Firefly定制主题，简洁美观。
1.  Xserver 使用 GPU + RGA 进行 2D 加速，运行流畅，占用 CPU 资源少。
1.  针对嵌入式平台，精简系统服务。
1.  提供基于 Arm Mali GPU 的 OpenGL、OpenCL 支持。
1.  提供基于 Rockchip VPU + Mpp 的视频硬编解码支持。
1.  适配 QT、Docker 等开发框架。
1.  提供一系列接口，以操作板载资源设备。
1.  系统采用 overlayfs 文件系统，支持导出 rootfs，二次打包，恢复出厂设置等功能


## 基础信息

### 用户和密码

Firefly Linux Desktop 系统开机启动后，自动登录到 firefly 用户。

如果有连接调试串口，串口终端自动登录root用户。

-   firefly 用户密码： firefly

-   root 用户：默认没有设置 root 密码，firefly 用户通过`sudo passwd root`命令自行配置 root 密码。

请参照 [《Firefly Ubuntu 使用手册》](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html)