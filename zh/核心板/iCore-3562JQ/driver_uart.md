# UART 使用

## 简介

AIO-3562JQ支持 UART、RS232、RS485 接口
* UARTx1
* RS485x2
* RS232x2

其中 UART 为 uart7，开发板的 RS232 接口由主控的 uart8 和 uart9 扩展出来，而 RS485 由 uart5 和 uart6 扩展出来。

AIO-3562JQ开发板的串口接口图如下：

![](../../../rk3562_img/iCore-3562JQ/uart_interface.jpg)

## DTS 配置

文件路径`kernel/arch/arm64/boot/dts/rockchip/rk3562-firefly-aio-3562jq.dtsi`
```
/* RS485 */
&uart5 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart5m1_xfer>;
};

&uart6 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart6m0_xfer>;
};

&uart7 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart7m0_xfer>;
};

/* RS232 */
&uart8 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart8m0_xfer>;
};

&uart9 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&uart9m1_xfer>;
};
```

硬件接口对应软件上的节点分别为：

```
485A1/B1：   /dev/ttyS5
485A2/B2：   /dev/ttyS6
TX7/RX7：    /dev/ttyS7
232TX1/RX1:  /dev/ttyS8
232TX2/RX2:  /dev/ttyS9
```

## 收发控制

RS485 需要额外控制收发，拓展 gpio 芯片 PCA9555 的最后一个引脚控制 485A1/B1 的收发，倒数第二引脚控制 485A2/B2 的收发。

首先查找 PCA9555 GPIO 的编号，执行如下命令可以看到编号范围是 496-511。该范围可能会因为修改内核而变化，以实际情况为准。
```
root@firefly#: cat /sys/kernel/debug/gpio | grep 2-0021
gpiochip5: GPIOs 496-511, parent: i2c/2-0021, 2-0021, can sleep:
```

那么最后一个引脚的编号为 511，倒数第二个为 510。

使用 /sys/class/gpio 子系统操作引脚：
```
# 申请使用 511 号引脚
echo 511 > /sys/class/gpio/export

# 设置方向为输出
echo out > /sys/class/gpio/gpio511/direction

# 写入 1 表示引脚输出高电平，控制 RS485 进入发送模式
echo 1 > /sys/class/gpio/gpio511/value

# 写入 0 表示引脚输出低电平，控制 RS485 进入接收模式
echo 0 > /sys/class/gpio/gpio511/value

# 另一个引脚也是同理，更换编号即可
```

上述操作可以通过代码读写文件完成。
