# Firefly Ubuntu Minimal介绍

AIO-3399C Ubuntu Minimal系统基于Ubuntu 64bit系统构建，目前有发布有Ubuntu18.04这个版本。与Ubuntu Desktop 相比具有以下特性：

1.  没有桌面环境，占用资源少，在简化网络管理之后，只需40M内存；
1.  针对嵌入式平台，精简系统服务。
1.  适配QT、Docker、Electron等开发框架。
1.  提供一系列接口，以操作板载资源设备。
1.  系统采用overlayfs文件系统，支持导出rootfs，二次打包，恢复出厂设置等功能

## 基础信息
1. Firefly Linux Minimal系统开机启动后，自动登录到root用户，密码为firefly
2. 系统已经添加OpenGL ES, OpenCL, DRM支持。

更多介绍，请参照 [《Firefly Ubuntu 使用手册》](ubuntu_manual.md)