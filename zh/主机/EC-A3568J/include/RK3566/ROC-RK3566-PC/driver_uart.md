# UART 使用

## 简介

Firefly-RK3399 支持五路 UART：UART0, UART1, UART2, UART3, UART4，都拥有两个 64 字节的 FIFO 缓冲区，用于数据接收和发送。 其中：

*    UART0 用于蓝牙传输，UART2 用作调试串口，只有 UART0 和 UART3 支持硬件自动流控。
*    支持比特率 115.2Kbps，460.8Kbps，921.6Kbps，1.5Mbps，3Mbps，4Mbps。
*    支持自选波特率，即使使用非整数时钟分频器
*    支持基于中断或基于 DMA 的模式
*    支持 5-8l 位宽度传输

我们 Firefly-RK3399 开发板为了方便用户使用，引出了一排通用的 GPIO，其对应原理图如下图：

![](../../img/Firefly-RK3399/uart_interface.jpg)

其中 GPIO1_A7 和 GPIO1_B0 两个 IO 口可复用为 `uart4_rx` 和 `uart4_tx`。

## DTS 配置

文件 `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi` 有 UART 相关节点的定义：

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

serial0 等串口在该文件的 `aliases` 节点中被定义为：`serial0 = &uart0;`。

因为我们 Firefly-RK3399 开发板引出了 uart4 供用户使用，所以这里就以 uart4 为例，介绍使用方法。下面是 uart4 节点相关定义：

```
uart4: serial@ff370000 {
	compatible = "rockchip,rk3399-uart", "snps,dw-apb-uart";
	reg = <0x0 0xff370000 0x0 0x100>;
	clocks = <&pmucru SCLK_UART4_PMU>, <&pmucru PCLK_UART4_PMU>;
	clock-names = "baudclk", "apb_pclk";
	interrupts = <GIC_SPI 102 IRQ_TYPE_LEVEL_HIGH 0>;
	reg-shift = <2>;
	reg-io-width = <4>;
	pinctrl-names = "default";
	pinctrl-0 = <&uart4_xfer>;
	status = "disabled";
};
uart4 {
	uart4_xfer: uart4-xfer {
	rockchip,pins =
		<1 7 RK_FUNC_1 &pcfg_pull_up>,
		<1 8 RK_FUNC_1 &pcfg_pull_none>;
        };
};
```

用户只需要在 `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-port.dtsi` 文件中使能该节点即可使用，如下：

```
&uart4 {
       current-speed = <9600>;
       no-loopback-test;
       status = "okay";
};
```

注意：由于 uart4_rx 和 uart4_tx 两个脚可复用为 spi1_rxd 和 spi1_txd，所以要留意关闭掉 spi1 的使用，如下：

```
&spidev0 {
	status = "disabled";
};
```

## 调试方法

配置好串口后，用户可以通过主机的 USB 转串口适配器向开发板的串口收发数据，步骤如下：

(1) 连接硬件

将开发板 UART4 的 TX、RX、GND 引脚分别和主机串口适配器的 TX、RX、GND 引脚相连。

(2) 打开主机的串口终端

在终端打开 kermit ,并设置波特率：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 USB 转串口适配器的设备文件
* 波特率与配置 DTS 节点中的 `current-speed` 属性相同

(3) 发送数据

uart4 的设备文件为 `/dev/ttyS4`。在设备上运行下列命令：

```
echo firefly uart4 test... > /dev/ttyS4
```

主机中的串口终端即可接收到字符串 "`firefly uart4 test..."

(4) 接收数据

首先在设备上运行下列命令：

```
cat /dev/ttyS4
```

然后在主机的串口终端输入字符串 "Firefly uart4 test..."，设备端即可见到相同的字符串。

## FAQs

###### Q1: 为何板子接上串口适配器后系统报错？

A1: Firefly RK3399 开发板的 TX 和 RX，分别对应串口适配器（官方）的 TX 和 RX，如果搞混淆了会导致通信出错。