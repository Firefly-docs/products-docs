# UART 使用

## 简介

AIO-1126BJD4V0 支持 UART3、UART4 接口。

## DTS配置

文件路径`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4v0.dts`

```dts
#ifdef FF_UART3
&uart3 {
        pinctrl-0 = <&uart3m1_xfer_pins>;
        status = "okay";
};
#endif

#ifdef FF_UART4
&uart4 {
        pinctrl-0 = <&uart4m1_xfer_pins>;
        status = "okay";
};
#endif
```
配置好串口后，硬件接口对应软件上的节点分别为：

```
UART3:  /dev/ttyS3
UART4:  /dev/ttyS4
```

## 收发验证

用户可以根据不同的接口使用不同的主机的 USB 转串口适配器向开发板的串口收发数据，例如 UART3 的调试步骤如下：

(1) 开发板发送, 主机接收

```
# 主机终端先执行:
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
cat /dev/ttyUSB0

# 开发板调试串口终端执行：
# /dev/ttyS3 为 UART3 节点
echo "firefly UART3 test..." > /dev/ttyS3
```

主机终端即可接收到字符串 "firefly UART3 test..."

(2) 主机发送，开发板接收

```
# 开发板调试串口终端先执行:
# /dev/ttyS3 为 UART3 节点
busybox  stty -echo -F /dev/ttyS3       # 关闭回显
cat /dev/ttyS3

# 主机终端执行：
# /dev/ttyUSB0 为 主机串口适配器 的节点，根据实际修改
echo "firefly UART3 test..." > /dev/ttyUSB0
```

开发板调试串口终端即可接收到字符串 "firefly UART3 test..."