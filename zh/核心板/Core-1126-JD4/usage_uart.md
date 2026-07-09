# UART 使用

## 简介

AIO-1126-JD4 支持 UART1、UART3、UART4接口
## DTS配置
文件路径`kernel/arch/arm/boot/dts/rv1126-firefly-rk809.dtsi`
```
&uart1 {
        status = "okay";
};

&uart3 {
        pinctrl-0 = <&uart3m2_xfer &uart3m2_ctsn &uart3m2_rtsn>;
        status = "okay";
};

&uart4 {
        pinctrl-0 = <&uart4m1_xfer>;
        status = "okay";
};
```
配置好串口后，硬件接口对应软件上的节点分别为：

```
UART1:  /dev/ttyS1
UART3:  /dev/ttyS3
UART4:  /dev/ttyS4
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 UART1 的调试步骤如下：

(1) 开发板发送, 主机接收

```
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS1 为 UART1 节点
echo "firefly UART1 test..." > /dev/ttyS1
```

主机终端即可接收到字符串 "firefly UART1 test..."

(2) 主机发送，开发板接收

```
# 开发板调试串口终端先执行:
# /dev/ttyS1 为 UART1 节点
busybox  stty -echo -F /dev/ttyS1       # 关闭回显
cat /dev/ttyS1

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
echo "firefly UART1 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly UART1 test..."