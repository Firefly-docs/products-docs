
# UART 使用


## 简介
为了方便用户开发，ROC-RK3328-PC 在板子的双排扩展接口上引出了 UART4（与 SPI1 复用）等两个通道。 

* UART4 为 TTL 电平接口
* UART4 最高支持波特率 691,200
* 每个子通道都具备收与发的独立FIFO（256 BYTE），FIFO 的中断可按用户需求进行编程触发点
* 具备子串口接收 FIFO 的超时中断
* 支持起始位错误的检测

## DTS 配置
以 UART4 配置为例，由于硬件上与 SPI1 冲突，所以需要先在设备树上关闭 `spi1` 节点,然后再打开 `uart4` 节点。
```
&spi1 {
    status = "disabled";
};
&uart4 {
    status = "okay";
};
```
## 调试方法
配置好串口后，硬件接口对应软件上的节点分别为：
```
UART4：/dev/ttyS4
```

-  连接硬件

    将 UART4 的 TX 与 RX 口与 USB 转串口适配器的 RX 和 TX 相连。

-  打开主机的串口终端

    在终端打开 kermit，并设置波特率：
    ```
    $ sudo kermit
    C-Kermit> set line /dev/ttyUSB0
    C-Kermit> set speed 1500000
    C-Kermit> set flow-control none
    C-Kermit> connect
    ```
    `/dev/ttyUSB0` 为 USB 转串口适配器的设备文件

- 发送数据

    UART4 的设备文件为 `/dev/ttyS4`。在设备上运行下列命令：
    关闭串口回显：
    ```
    busybox stty -echo  -F /dev/ttyS4
    ```
    发送数据：
    ```
    echo firefly test... > /dev/ttyS4
    ```

    主机中的串口终端即可接收到字符串 “firefly test...”

- 接收数据

    首先在设备上运行下列命令：
    ```
    cat /dev/ttyS0
    ```
    然后在主机的串口终端输入字符串 “firefly test...”，设备端即可见到相同的字符串。