# 串口调试

Debug 串口在调试和排查问题时非常有用，特别是在图形界面不可用的情况下。

## 连接

使用 USB 线连接设备上的 Console 口和电脑：

![](../../../qcom_img/AIBOX-9075/debug_console.jpg)

## 驱动安装

我们使用的串口转 USB 芯片是 CH342。

Linux 电脑无需安装驱动。

Windows 电脑需要安装驱动，前往 [下载地址](https://www.t-firefly.com/doc/download/407.html#other_924) 下载 `CH343SER.EXE`

双击运行，点击安装驱动即可。

## 串口使用

驱动安装成功后，在 Windows 的设备管理器中应该可以看到名称为 "USB-Enhanced-SERIAL-A CH342" 和 "USB-Enhanced-SERIAL-B CH342"的两个串口设备。

其中 SERIAL-A 是主系统（Linux）的 Debug 串口，SERIAL-B 是子系统（RTOS）的 Debug 串口。

Linux 中则是 /dev/ttyUSBX 或 /dev/ttyACMX，其中数字 X 可能不同，可以拔插 USB 来寻找对应的设备。

找到设备后就可以使用你常用的串口工具 (MobaXterm、Minicom 等) 打开串口设备，波特率为 115200，数据位 8 位，停止位 1 位，无奇偶校验。