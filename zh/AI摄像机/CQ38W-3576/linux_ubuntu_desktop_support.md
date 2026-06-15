# Firefly Ubuntu Desktop介绍

CQ38W-3576 Ubuntu Desktop系统基于Ubuntu 64bit系统构建，目前支持Ubuntu18.04版本。具有以下特性：

1.  桌面环境采用LXDE，Lubuntu+Firefly定制主题，简洁美观。
1.  Xserver使用GPU + RGA进行2D加速，运行流畅，占用CPU资源少。
1.  针对嵌入式平台，精简系统服务。
1.  提供基于Arm Mali GPU的OpenGL、OpenCL支持。
1.  提供基于Rockchip VPU + Mpp 的视频硬编解码支持。
1.  适配QT、Docker、Electron等开发框架。
1.  提供一系列接口，以操作板载资源设备。
1.  系统采用overlayfs文件系统，支持导出rootfs，二次打包，恢复出厂设置等功能


## 基础信息

### 用户和密码

Firefly Linux Desktop系统开机启动后，自动登录到firefly用户。

如果有连接调试串口，串口终端自动登录root用户。

-   firefly用户密码： firefly

-   root用户：默认没有设置root密码，firefly用户通过`sudo passwd root`命令自行配置root密码。

更多介绍，请参照 [《Firefly Ubuntu 使用手册》](linux_ubuntu_manual.md)
