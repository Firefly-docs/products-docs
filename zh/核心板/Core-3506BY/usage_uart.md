# UART 使用

## 硬件

ROC-RK3506B-CC 硬件版本的串口 5 接口图如下：

![](../../../rk3506_img/ROC-RK3506B-CC/usage_uart_interface.jpg)

## DTS配置

UART5 默认没有打开。在文件路径`kernel/arch/arm/boot/dts/rk3506b-firefly-roc-rk3506b-cc.dtsi` 末尾添加以下内容并且编译内核烧录到板子：
```
/* uart5 */
&uart5 {
	pinctrl-0 = <&uart5m1_xfer_pins &uart5m1_ctsn_pins &uart5m1_rtsn_pins>;
	status = "okay";
};
```

配置好串口后，硬件接口对应软件上的节点为：
```
UART5:   /dev/ttyS5
```

## UART 收发验证

最简单的方式短接 UART5 TX RX 引脚, 然后使用命令在调试串口或 ADB 执行命令

```
busybox  stty -echo -F /dev/ttyS5         # 关闭回显
cat /dev/ttyS5 &                           # 后台获取/dev/ttyS5输入字符串
echo "firefly uart test..." > /dev/ttyS5  # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."