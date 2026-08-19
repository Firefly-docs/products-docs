# 升级前准备

## 前言

本文介绍了如何将固件，通过 Type-A 转 C USB 数据线 烧录到 AIBOX-8550 开发板的存储器中。

## 准备工具
* AIBOX-8550 开发板
* 处理器架构为 X86_64 的 windows 电脑
* 良好的 Type-A 转 C USB 数据线

## 准备固件
固件可以通过编译SDK获得，也可以通过[资源下载](https://www.t-firefly.com/doc/download/356.html)处下载公版固件（完整固件）。

* 完整固件

    完整固件是由 parameter、bootloader、kernel、system 等所有文件打包合并成的单个文件。完整固件会是一个压缩包，里面包含很多文件。Firefly 正式发布的固件都是采用统一固件格式，升级统一固件将会更新主板上所有分区的数据和分区表，并且擦除主板上所有数据。

* 分区镜像
    
    分区镜像只包含单个系统分区的内容，分区镜像是单个文件。最常见的就是 boot.img 和 dtbo.img，开发过程中可以只单独烧录某个分区镜像来验证修改、部分更新、对比测试等。烧录分区镜像的前提是，设备本身需要正常运行。若设备无法开机，则只能烧录完整固件。

## 安装烧写工具
* 安装 USB 驱动

前往 [USB 驱动](https://www.t-firefly.com/doc/download/356.html#other_924) 下载驱动。

有两份驱动，先安装高通 usb 驱动： Qualcomm USB Driver，双击 exe 文件运行，接受用户协议一直点 next 即可，最后点 finish 安装完成。

再安装 Google USB 驱动，解压 usb_driver_r13-windows，右键里面的 android_winusb.inf，点击安装，一直点确认即可。

* 安装 QTSP

前往 [QTSP 工具](https://www.t-firefly.com/doc/download/356.html#other_925) 下载工具。

解压后，双击 QPST.2.7.496.1.exe 开始安装。接受用户协议一直点 next 即可，最后点 install 安装，点 finish 安装完成。

安装完成后，我们要用到的工具位于 C:\Program Files (x86)\Qualcomm\QPST\bin\QFIL.exe

双击 QFIL.exe 打开工具。

* 安装 Android SDK Platform-Tools

前往 [Platform Tools](https://www.t-firefly.com/doc/download/356.html#other_923) 下载工具。

下载后解压即可，内含 adb.exe 和 fastboot.exe