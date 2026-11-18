# Ethernet 使用

## 简介 [step]

Firefly-R3572A 提供 2 个 RJ45 千兆以太网接口（1000 Mbps），均支持自适应交叉线序。

## DTS 配置 [step]

GMAC 控制器节点由 SDK 在芯片级 DTS 中定义，板级配置包括 PHY 复位引脚、时钟延迟（tx_delay/rx_delay）与 pinctrl 引脚复用等。以下 gmac0 配置取自 Firefly-R3572A SDK 的板级 DTS（`rk3572-firefly-r3572a.dtsi`），gmac1 结构类似（复位脚为 gpio4 PB4，对应 `&eth1m0_*_pins`）：

```text
&gmac0 {
    /* Use rgmii-rxid mode to disable rx delay inside Soc */
    phy-mode = "rgmii-rxid";
    clock_in_out = "output";
    snps,reset-gpio = <&gpio0 RK_PB6 GPIO_ACTIVE_LOW>;
    snps,reset-active-low;
    /* Reset time is 20ms, 100ms for rtl8211f */
    snps,reset-delays-us = <0 20000 100000>;
    pinctrl-names = "default";
    pinctrl-0 = <&gmac0_miim
            &gmac0_tx_bus2
            &gmac0_rx_bus2
            &gmac0_rgmii_clk
            &gmac0_rgmii_bus>;
    tx_delay = <0x20>;
    phy-handle = <&rgmii_phy0>;
    status = "okay";
};
```

2 个网口分别对应两个 GMAC 控制器节点，管脚复用与延时参数需要根据实际硬件测量调整。

## 查看网络接口 [step]

系统启动后查看网卡状态：

```bash
ifconfig -a
```

2 个网口默认命名为 `eth0`、`eth1`，命名顺序以系统实际枚举为准。查看协商速率：

```bash
ethtool eth0
```

`Speed` 显示 `1000Mb/s` 表示已工作在千兆模式。

## 获取 IP 地址 [step]

网口默认通过 DHCP 获取地址，也可手动配置：

```bash
udhcpc -i eth0                          # DHCP 获取
ifconfig eth0 192.168.1.100 netmask 255.255.255.0   # 静态配置
```

## 连通性测试 [step]

分别对两个网口做连通性测试：

```bash
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 www.baidu.com
```

正常情况下无丢包，`rtt` 时间与网络环境相关。

## 硬件连接 [step]

RJ45 接口内置网络变压器，使用标准网线连接路由器或交换机即可；接口位于主板边缘，标注位置见 [介绍](../product_introduction/started.md) 的接口图。
