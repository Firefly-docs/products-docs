
# UART 使用

## 简介

ROC-RK3399-PC 支持2路UART：UART0, UART2，每路UART都拥有两个64字节的FIFO缓冲区，用于数据接收和发送。 其中：

* UART0已引出供使用者开发，UART2用作调试串口；
* 支持比特率115.2Kbps，460.8Kbps，921.6Kbps，1.5Mbps，3Mbps，4Mbps；
* 支持自选波特率，即使使用非整数时钟分频器；
* 支持基于中断或基于DMA的模式；
* 支持5-8位宽度传输。

我们 ROC-RK3399-PC 开发板为了方便用户使用，引出了一排通用的GPIO，其对应原理图如下图：
![](../../../rk3399_img/Station-P1-Pro/roc-rk3399-pc7.jpg)
![](../../../rk3399_img/Station-P1-Pro/roc-rk3399-pc-e.jpg)


## DTS配置

文件kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi 有UART相关节点的定义：
```
aliases {
...
serial0 = &uart0;
serial1 = &uart1;
serial2 = &uart2;
serial3 = &uart3;
serial4 = &uart4;
};
```

serial0等串口在该文件的 aliases 节点中被定义为：serial0 = &uart0;

因为我们 ROC-RK3399-PC 开发板引出了uart4供用户使用，所以这里就以uart0为例，介绍使用方法。下面是uart4节点相关定义：

```
uart0: serial@ff180000 {
	compatible = "rockchip,rk3399-uart";
	reg = <0x0 0xff180000 0x0 0x100>;
	clocks = <&cru SCLK_UART0>, <&cru PCLK_UART0>;
	clock-names = "baudclk", "apb_pclk";
	interrupts = <GIC_SPI 99 IRQ_TYPE_LEVEL_HIGH 0>;
	reg-shift = <2>;
	reg-io-width = <4>;
	pinctrl-names = "default";
	pinctrl-0 = <&uart0_xfer &uart0_cts &uart0_rts>;
	status = "disabled";
};
...
uart0 {
	uart0_xfer: uart0-xfer {
		rockchip,pins =
			<2 16 RK_FUNC_1 &pcfg_pull_up>,
			<2 17 RK_FUNC_1 &pcfg_pull_none>;
	};
};
...
```

用户只需要在kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-port.dtsi文件中使能该节点即可使用，如下：

```
&uart0 {
       current-speed = <9600>;
       no-loopback-test;
       status = "okay";
};
```

调试方法

配置好串口后，用户可以通过主机的 USB 转串口适配器向开发板的串口收发数据，步骤如下：

(1) 连接硬件

将开发板 UART0 的 TX、RX、GND 引脚分别和主机串口适配器的 TX、RX、GND 引脚相连(因我们开发板已经将TX和RX反转过，所以连接时一一对应就行)。

(2) 打开主机的串口终端

在终端打开kermit,并设置波特率：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* /dev/ttyUSB0 为 USB 转串口适配器的设备文件

* 波特率与配置 DTS 节点中的 current-speed 属性相同


(3) 发送数据

uart0 的设备文件为 /dev/ttyS0。在设备上运行下列命令：


```
echo firefly uart0 test... > /dev/ttyS0
```

主机中的串口终端即可接收到字符串“firefly uart0 test…”

(4) 接收数据

首先在设备上运行下列命令：

```
cat /dev/ttyS0
```

然后在主机的串口终端输入字符串 “Firefly uart0 test…”，设备端即可见到相同的字符串。


## FAQs
**Q1: 为何板子接上串口适配器后系统报错？**

A1:ROC-RK3399-PC 开发板的TX和RX，分别对应串口适配器（官方）的TX和RX，如果搞混淆了会导致通信出错。
