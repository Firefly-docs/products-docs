

# UART 使用

## 简介

AIO-3399C 支持 SPI 桥接/扩展 4 个增强功能串口（UART）的功能，分别为 UART1，UART2，RS232，RS485。每个 UART 都拥有 256 字节的 FIFO 缓冲区，用于数据接收和发送。其中：

* UART1，UART2 为 TTL 电平接口，RS232 为 RS232 电平接口，RS485 为 RS485 电平接口

* 每个子通道 UART 的波特率、字长、校验格式可以独立设置,最高可以提供 2Mbps 的通信速率
* 每个子通道具备收/发独立的 256 BYTE FIFO，FIFO 的中断可按用户需求进行编程触发点
* 具备子串口接收 FIFO 超时中断
* 支持起始位错误检测

AIO-3399C 开发板的串口接口图如下：

![](../../../rk3399_img/AIO-3399C/uart_interface.jpg)

**注意:RS232 需要使用RS232交叉线才能使用，不然rx和tx会出现收发不成功的问题**
## DTS 配置

文件 `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-port.dtsi` 有 SPI 转 UART 相关节点的定义：

```
&spi1 {
    spi_wk2xxx: spi_wk2xxx@00{
    status = "disabled";
    compatible = "firefly,spi-wk2xxx";
    reg = <0x00>;
    spi-max-frequency = <10000000>;
    power-gpio = <&gpio2 4 GPIO_ACTIVE_HIGH>;
    reset-gpio = <&gpio1 17 3 GPIO_ACTIVE_HIGH>;
    irq-gpio = <&gpio1 2 IRQ_TYPE_EDGE_FALLING>;
    cs-gpio = <&gpio1 10 GPIO_ACTIVE_HIGH>;
    /* rk3399 driver support SPI_CPOL | SPI_CPHA | 			SPI_CS_HIGH */
    //spi-cpha;     /* SPI mode: CPHA=1 */
    //spi-cpol;     /* SPI mode: CPOL=1 */
    //spi-cs-high;
     };
}
```

可以看到，在 `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-aioc.dts` 文件中使能该节点即可使用。另外，由于我们板子使用的 SPI 转 UART 串口模块挂到 SPI1 上，所以还要使能 SPI1 节点。如下：

```
&spi1 {
    status = "okay";
};

&spi_wk2xxx {
    status = "okay";
};
```

注意：由于 `spi1_rxd` 和 `spi1_txd` 两个脚可复用为 `uart4_rx` 和 `uart4_tx`，所以要留意关闭掉 uart4 的使用，如下：

```
&uart4 {
    status = "disabled";
};
```

## 调试方法

配置好串口后，硬件接口对应软件上的节点分别为：

```
RS485：/dev/ttysWK0
RS232：/dev/ttysWK1
UART2：/dev/ttysWK2
UART1：/dev/ttysWK3
```

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

将开发板RS485 的A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

(2) 打开主机的串口终端

在终端打开 kermit，并设置波特率：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 USB 转串口适配器的设备文件

(3) 发送数据

RS485 的设备文件为 /dev/ttysWK0。在设备上运行下列命令：

```
echo firefly RS485 test... > /dev/ttysWK0
```

主机中的串口终端即可接收到字符串 "firefly RS485 test..."

(4) 接收数据

首先在设备上运行下列命令：

```
cat /dev/ttysWK0
```

然后在主机的串口终端输入字符串 "Firefly RS485 test..."，设备端即可见到相同的字符串。


