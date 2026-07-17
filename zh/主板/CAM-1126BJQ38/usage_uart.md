# UART 使用

## 简介

ICORE-1126BQ38 底板引出了一个 RS485 接口。

## DTS配置

文件路径`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-cam-1126bq38.dtsi`

```dts
#ifdef FF_RS485
&uart3 {
        pinctrl-names = "default";
        pinctrl-0 = <&uart3m1_xfer_pins>;
        status = "okay";
};
#endif
```

配置好串口后，硬件接口对应软件上的节点分别为：

```
UART3:  /dev/ttyS3        RS485
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转 RS485 适配器向开发板的 RS485 接口收发数据，例如 UART3 的调试步骤如下：

(1) 开发板发送, 主机接收

```bash
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器的节点，根据实际修改。波特率设置为 115200 。
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS3 为 UART3 RS485 节点。波特率设置为 115200 。
stty -F /dev/ttyS3 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART3 test..." > /dev/ttyS3
```

主机终端即可接收到字符串 "firefly UART3 test..."

(2) 主机发送，开发板接收

```bash
# 开发板调试串口终端先执行:
# /dev/ttyS3 为 UART3 RS485 节点。波特率设置为 115200 。
stty -F /dev/ttyS3 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyS3

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器的节点，根据实际修改。波特率设置为 115200 。
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART3 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly UART3 test..."