# UART 使用

## 板载资源介绍

Firefly-RK3128 开发板内置 3 路 UART，分别为 uart0，uart1，uart2。  
uart0 用于蓝牙数据传输，如果要使用 uart0，必须关掉蓝牙，才可以使用扩展槽上的 UART0 针脚。  
uart1, 因为存在以下复用：

* BT_HOST_WAKE/SPI_TXD/UART1_TX  
* BT_WAKE/SPI_RXD/UART1_RX  
* WIFI_REG_ON/SPI_CSN0/UART1_RTS  


若要使用 uart1, 必须关掉蓝牙和 SPI 功能，这样才可以使用扩展槽上的 SPI_RX 和 SPI_TX 针作 UART1_RX 和 UART1_TX 使用。   
uart2 一般用做调试串口，但同样存在复用，也就是说 TF 卡与调试串口不可以同时使用： 

* SDMMC_D0/UART2_TX  
* SDMMC_D1/UART2_RX  


uart1 和 uart2 有 32 字节的 FIFO 收发缓冲区，uart0 则要有双 64 字节的 FIFO 用作蓝牙数据收发。所有 uart 均支持 5 位、6 位、7 位、8 位数据收发和 DMA 操作。  

## 配置 DTS 节点

文件 kernel/arch/arm/boot/dts/rk312x.dtsi 中已经有 uart 相关节点定义，如下所示：

```
uart0: serial@20060000 {
  compatible = "rockchip,serial";
  reg = <0x20060000 0x100>;
  interrupts = <GIC_SPI 20 IRQ_TYPE_LEVEL_HIGH>;
  clock-frequency = <24000000>;
  clocks = <&clk_uart0>, <&clk_gates8 0>;
  clock-names = "sclk_uart", "pclk_uart";
  reg-shift = <2>;
  reg-io-width = <4>;
  dmas = <&pdma 2>, <&pdma 3>;#dma-cells = <2>;
  pinctrl-names = "default";
  pinctrl-0 = <&uart0_xfer &uart0_cts &uart0_rts>;
  status = "disabled";
}; 
uart1: serial@20064000 {
  compatible = "rockchip,serial";
  reg = <0x20064000 0x100>;
  interrupts = <GIC_SPI 21 IRQ_TYPE_LEVEL_HIGH>;
  clock-frequency = <24000000>;
  clocks = <&clk_uart1>, <&clk_gates8 1>;
  clock-names = "sclk_uart", "pclk_uart";
  reg-shift = <2>;
  reg-io-width = <4>;
  dmas = <&pdma 4>, <&pdma 5>;#dma-cells = <2>;
  pinctrl-names = "default";
  pinctrl-0 = <&uart1_xfer &uart1_cts &uart1_rts>;
  status = "disabled";
};
```

### 配置 uart0

用户只需在 kernel/arch/arm/boot/dts/rk3128-fireprime.dts 文件中打开 uart0 ，并关掉蓝牙，如下所示：

```
&uart0 {
  status = "okay";
  dma-names = "!tx", "!rx";
  pinctrl-0 = <&uart0_xfer &uart0_cts>;}; 
  wireless-bluetooth {
  compatible = "bluetooth-platdata";
  ...
  status = "disabled";
};
```

### 配置 uart1

用户只需在 kernel/arch/arm/boot/dts/rk3128-fireprime.dts 文件中打开 uart1 ，并关掉蓝牙和 SPI，如下所示：

```
//...
wireless-bluetooth {
  compatible = "bluetooth-platdata";
  ...
  status = "disabled";
};//...
&spi0 {  
  status = "disabled";
};
&uart1 {
  status = "okay";
  dma-names = "!tx", "!rx";
  pinctrl-0 = <&uart1_xfer &uart1_cts>;
};
```

## 编译并烧写内核

将串口驱动编译到内核中，在 kernel 目录下执行如下命令：

```
make firefly_defconfig
make rk3128-fireprime.img -j4
```

把 kernel 目录下生成的 kernel.img 和 resource.img 烧录到开发板中即可。

## 测试串口通讯

配置好串口后，用户可以通过主机的 USB 转串口适配器向开发板的串口收发数据，以 uart0 为例，步骤如下：

### (1) 连接硬件

将开发板 uart0 的 TX、RX、GND 引脚分别和主机串口适配器的 RX、TX、GND 引脚相连。     
注意：如果是使用 Firefly 的串口适配器，则是 TX 对 TX，RX 对 RX 连接。 

### (2) 打开主机的串口终端

在终端打开 kermit，并设置波特率：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* /dev/ttyUSB0 为 USB 转串口适配器的设备文件
* uart0 的波特率默认为 9600


### (3) 发送数据

uart0 的设备文件为 /dev/ttyS0。在设备上运行下列命令：

```
echo firefly uart test... > /dev/ttyS0
```

主机中的串口终端即可接收到字符串“firefly uart test...”

### (4) 接收数据

首先在设备上运行下列命令：

```
cat /dev/ttyS0
```

然后在主机的串口终端输入字符串 “Firefly uart test...”，设备端即可见到相同的字符串。   
要改变 uart0 的波特率，例如 115200，可以运行以下命令：   
```
stty -F /dev/ttyS0 115200
```