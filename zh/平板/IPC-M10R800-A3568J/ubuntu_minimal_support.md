# Firefly Ubuntu Minimal 介绍

IPC-M10R800-A3568J Ubuntu Minimal 系统基于 Ubuntu 64bit 系统构建，目前主要有 Ubuntu20.04 这个版本。与 Ubuntu Desktop 相比具有以下特性：

1.  没有桌面环境，占用资源少，在简化网络管理之后，只需 40M 内存；
1.  针对嵌入式平台，精简系统服务。
1.  适配 QT、Docker 等开发框架。
1.  提供一系列接口，以操作板载资源设备。
1.  系统采用 overlayfs 文件系统，支持导出 rootfs，二次打包，恢复出厂设置等功能

## 基础信息

1. Firefly Linux Minimal 系统开机启动后，自动登录到 root 用户，密码为 firefly
2. 系统已经添加 OpenGL ES, OpenCL, DRM 支持。

请参照 [《Firefly Ubuntu 使用手册》](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_ubuntu.html)