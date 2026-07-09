# UART 使用

## 板载资源介绍

Firefly-RK3288  开发板内置 5 路 UART，分别为 uart0，uart1，uart2，uart3，uart4。

* uart0 为 uart_bt，用于蓝牙传输。
* uart2 为 uart_dbg，用做调试串口。
* uart 1、uart3、uart4 可做外部串口使用，开发板已将其引脚连接至 J10 处，其中 uart4 和 SPI0 引脚复用。

拥有 64 字节的 FIFO 收发缓冲区，支持 5 位、6 位、7 位、8 位数据收发和 DMA 操作。

## 配置步骤

以下以配置 uart3 为例。

### 配置 DTS 节点

文件 `kernel/arch/arm/boot/dts/rk3288.dtsi` 中已经有 uart 相关节点定义，如下所示：

```dts
uart_gps: serial@ff1b0000 {
    compatible = "rockchip,serial";
    reg = <0xff1b0000 0x100>;
    interrupts = <GIC_SPI 58 IRQ_TYPE_LEVEL_HIGH>;
    clock-frequency = <24000000>;
    clocks = <&clk_uart3>, <&clk_gates6 11>;
    clock-names = "sclk_uart", "pclk_uart";
    current-speed = <115200>;
    reg-shift = <2>;
    reg-io-width = <4>;
    dmas = <&pdma1 7>, <&pdma1 8>;#dma-cells = <2>;
    pinctrl-names = "default";
    pinctrl-0 = <&uart3_xfer &uart3_cts &uart3_rts>;
    status = "disabled";
};
```

注：uart_gps 在该文件的 aliases 节点中被定义为：serial3 = &uart_gps；

用户只需在 `kernel/arch/arm/boot/dts/firefly-rk3288.dts` 文件中打开所要使用的节点即可，如下所示：

```dts
&uart_gps {
    status = "okay";
    dma-names = "!tx", "!rx";pinctrl-0 = <&uart3_xfer &uart3_cts>;
};
```

### 编译并烧写内核

将串口驱动编译到内核中，在 kernel 目录下执行如下命令：

```bash
make firefly-rk3288.img
```

把 kernel 目录下生成的 kernel.img 和 resource.img 烧录到开发板中即可。

### 串口通讯

配置好串口后，用户可以通过主机的 USB 转串口适配器向开发板的串口收发数据，步骤如下：

1. 连接硬件

   将开发板 uart3 的 TX、RX、GND 引脚分别和主机串口适配器的 RX、TX、GND 引脚相连。

2. 打开主机的串口终端

   在终端打开kermit,并设置波特率：

```bash
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 115200
C-Kermit> set flow-control none
C-Kermit> connect
```

`/dev/ttyUSB0` 为 USB 转串口适配器的设备文件，波特率与配置 DTS 节点中的 current-speed 属性相同。

3. 发送数据：

   uart3 的设备文件为 `/dev/ttyS3`。在设备上运行下列命令：

```bash
echo firefly uart3 test... > /dev/ttyS3
```

4. 接收数据

首先在设备上运行下列命令：

```bash
cat /dev/ttyS3
```

然后在主机的串口终端输入字符串 “Firefly uart3 test...”，设备端即可见到相同的字符串。