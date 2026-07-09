# UART 使用

## 简介

AIO-1126BQ38 支持 UART3、UART5 接口。其中 UART3 用作 RS485 ，UART5 用作 RS232 。

## DTS配置

文件路径`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bq38.dts`

```dts
#ifdef FF_UART3
&uart3 {
        pinctrl-0 = <&uart3m1_xfer_pins>;
        status = "okay";
};
#endif

#ifdef FF_UART5
&uart5 {
        pinctrl-0 = <&uart5m1_xfer_pins>;
        status = "okay";
};
#endif
```
配置好串口后，硬件接口对应软件上的节点分别为：

```
UART3:  /dev/ttyS3	RS485
UART5:  /dev/ttyS5	RS232
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 UART5 的调试步骤如下：

(1) 开发板发送, 主机接收

```bash
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器的节点，根据实际修改。波特率设置为 115200 。
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS5 为 UART5 节点。波特率设置为 115200 。
stty -F /dev/ttyS5 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART5 test..." > /dev/ttyS5
```

主机终端即可接收到字符串 "firefly UART5 test..."

(2) 主机发送，开发板接收

```bash
# 开发板调试串口终端先执行:
# /dev/ttyS5 为 UART5 节点。波特率设置为 115200 。
stty -F /dev/ttyS5 115200 cs8 -cstopb -parenb -icanon -echo
cat /dev/ttyS5

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器的节点，根据实际修改。波特率设置为 115200 。
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -parenb -icanon -echo
echo "firefly UART5 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly UART5 test..."


## RS485

其中 GPIO0_B0 用作 RS485 控制收发引脚，该引脚拉高是发送，拉低为接收。用户可以通过 /sys/class/gpio 子系统去操作。

* 下面是 GPIO0_B0 配置成输出

```bash
echo 8 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio8/direction
```

* GPIO0_B0 拉高，RS485 发送： echo 1 > /sys/class/gpio/gpio8/value
* GPIO0_B0 拉低，RS485 接收： echo 0 > /sys/class/gpio/gpio8/value