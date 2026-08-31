# Windows 驱动安装

## 安装驱动

我们的串口模块使用的是 CP2104，所以下载驱动并安装:

* [CP210X](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers)

如果您另外购买了使用其他芯片的模块，比如 CH340 或 PL2303，可以从这下载驱动：

* [CH340](https://www.wch.cn/downloads/CH341SER_EXE.html)
* [PL2303](https://www.prolific.com.tw/en/portfolio-item/pl2303gl/)

插入适配器后，系统会提示发现新硬件，并初始化，之后可以在设备管理器找到对应的 COM 口：

![](../../../modules_img/USB-TO-TTL-Serial/debug_find_com.png)