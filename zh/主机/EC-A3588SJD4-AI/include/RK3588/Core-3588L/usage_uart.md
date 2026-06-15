# UART 使用

## 硬件

AIO-3588SJD4-AI 硬件版本的串口接口图如下：

![](../../../rk3588_img/EC-A3588SJD4-AI/usage_uart_interface.png)

## DTS配置

文件路径`kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588l.dtsi`
其中UART3可配置为CAN1,默认公版sdk都是配置为UART3,如果需要配置为CAN1,只需要配置CAN1_OR_UART3为1
```
#define CAN1_OR_UART3 0 /*1 = CAN1 , 0 = UART3 */
...
...
...
//ext gpio
&uart6 {
    pinctrl-names = "default";
    pinctrl-0 = <&uart6m1_xfer &uart6m1_ctsn &uart6m1_rtsn>;
    status = "okay";
};

&uart7 {
    pinctrl-names = "default";
    pinctrl-0 = <&uart7m1_xfer>;
    status = "okay";
};

&uart8 {
    pinctrl-names = "default";
    pinctrl-0 = <&uart8m1_xfer>;
    status = "okay";
};

#if CAN1_OR_UART3
&can1 {
    status = "okay";
    assigned-clocks = <&cru CLK_CAN1>;
    assigned-clock-rates = <200000000>;
    pinctrl-names = "default";
    pinctrl-0 = <&can1m0_pins>;
};
#else
&uart3 {
    pinctrl-names = "default";
    pinctrl-0 = <&uart3m1_xfer>;
    status = "okay";
};
#endif
```

配置好串口后，硬件接口对应软件上的节点为：
```
UART3:   /dev/ttyS3
UART6:   /dev/ttyS6
UART7:   /dev/ttyS7
UART8:   /dev/ttyS8
```

## UART 收发验证

最简单的方式短接UART7 TX RX 引脚, 然后使用命令在调试串口或ADB执行命令

```
busybox  stty -echo -F /dev/ttyS7          # 关闭回显
cat /dev/ttyS7 &                           # 后台获取/dev/ttyS7输入字符串
echo "firefly uart test..." > /dev/ttyS7   # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."
