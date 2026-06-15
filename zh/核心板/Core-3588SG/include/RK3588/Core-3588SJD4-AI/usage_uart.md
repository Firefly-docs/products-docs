# UART 使用

## 简介

AIO-3588SG 支持 RS232、RS485 接口

AIO-3588SG 开发板的串口接口图如下：

![](../../../rk3588_img/Core-3588SG/uart_interface.jpg)

## DTS配置
开发板的 RS232 接口由主控的 UART9 扩展出来，而 RS485 接口由主控 UART6 扩展出来。

文件路径`kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588sjd4-ai.dtsi
```
&uart6 {
        pinctrl-0 = <&uart6m1_xfer>;
        status = "okay";
};

&uart9 {
        pinctrl-0 = <&uart9m2_xfer>;
        status = "okay";
};
```
配置好串口后，硬件接口对应软件上的节点分别为：

```
RS232 :   /dev/ttyS9
RS485 :   /dev/ttyS6
```

## 收发验证

引脚 GPIO1_A2 用于 RS485 的收发控制，该引脚拉高是发送，拉低为接收。用户可以通过 /sys/class/gpio 子系统去操作。

串口默认波特率都是 9600，数据位 8 位，停止位 1 位，无流控。可使用 echo 和 cat 命令进行简单的收发测试。