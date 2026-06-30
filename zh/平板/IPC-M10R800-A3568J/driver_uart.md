

# UART 使用

## 简介

AIO-3568J支持UART、RS232、RS485接口
* UARTx2
* RS485x2
* RS232x2

其中开发板的RS232接口由主控的uart3和uart4扩展出来，而UART和RS485则由SPI 桥接/扩展芯片wk2124扩展出来，该芯片特性如下

* 每个子通道 UART 的波特率、字长、校验格式可以独立设置,最高可以提供 2Mbps 的通信速率
* 每个子通道具备收/发独立的 256 BYTE FIFO，FIFO 的中断可按用户需求进行编程触发点
* 具备子串口接收 FIFO 超时中断
* 支持起始位错误检测

AIO-3568J开发板的串口接口图如下：

![](../../../rk356x_img/IPC-M10R800-A3568J/uart_interface.png)

## DTS配置

文件路径`kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi`
```
&spi1 {
        status = "disabled";
        max-freq = <48000000>;
        dev-port = <0>;
        pinctrl-0 = <&spi1m1_pins>;
        pinctrl-1 = <&spi1m1_pins_hs>;

        spi_wk2xxx: spi_wk2xxx@00{
                status = "disabled";
                compatible = "firefly,spi-wk2xxx";
                reg = <0x00>;
                spi-max-frequency = <10000000>;
                power-gpio = <&pca9555 PCA_IO1_7 GPIO_ACTIVE_HIGH>;
                reset-gpio = <&pca9555 PCA_IO1_1 GPIO_ACTIVE_HIGH>;
                irq-gpio = <&gpio0 RK_PA6 IRQ_TYPE_EDGE_FALLING>;
                cs-gpio = <&gpio3 RK_PA1 GPIO_ACTIVE_HIGH>;
        };
};

```

可以看到，在 `kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dtsi` 文件中使能该节点即可使用。另外，由于我们板子使用的 SPI 转 UART 串口模块挂到 SPI1 上，所以还要使能 SPI1 节点。如下：

```

&uart3 {
        status = "okay";
};

&uart4 {
        status = "okay";
};

&spi1 {
        status = "okay";
};

&spi_wk2xxx {
        status = "okay";
};

```

配置好串口后，硬件接口对应软件上的节点分别为：

```
RS485_1：  /dev/ttysWK0
RS485_2：  /dev/ttysWK1
UART1：    /dev/ttysWK2
UART2：    /dev/ttysWK3
RS232_1 :  /dev/ttyS3
RS232_2:   /dev/ttyS4
```


## 调试方法

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 RS485 的调试步骤如下：

(1) 连接硬件

将开发板RS485 的A、B、GND 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B、GND 引脚相连。

(2) 打开主机的串口终端

在终端打开 kermit，设置波特率并连接：

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` 为 主机USB 转串口适配器的设备文件
*  ubuntu安装kermit命令`sudo apt install ckermit`

(3) 开发板发送数据

开发板的RS485 设备文件为 /dev/ttysWK0。在开发板设备上运行下列命令：

```
echo "firefly RS485 test..." > /dev/ttysWK0
```

主机中的串口终端即可接收到字符串 "firefly RS485 test..."

(4) 开发板接收数据

首先在开发板设备上运行下列命令：

```
cat /dev/ttysWK0
```

然后在主机的串口终端输入字符串 "Firefly RS485 test..."，设备端即可见到相同的字符串。

(5) 主机退出kermit串口连接

`ctrl+\`后按c，退回终端可输入exit

```
C-Kermit>exit
OK to exit? ok
```


