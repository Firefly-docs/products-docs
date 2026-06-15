## Ethernet 使用

### dts 配置

#### 公共的配置
`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-diff.dtsi`
```
&gmac0 {
    /* Use rgmii-rxid mode to disable rx delay inside Soc */
    phy-mode = "rgmii-rxid";
    clock_in_out = "output";

    snps,reset-gpio = <&gpio3 RK_PC7 GPIO_ACTIVE_LOW>;
    snps,reset-active-low;
    /* Reset time is 20ms, 100ms for rtl8211f */
    snps,reset-delays-us = <0 20000 100000>;

    pinctrl-names = "default";
    pinctrl-0 = <&gmac0_miim
            &gmac0_tx_bus2
            &gmac0_rx_bus2
            &gmac0_rgmii_clk
            &gmac0_rgmii_bus>;

    tx_delay = <0x45>;
    //rx_delay = <0x4a>;

    phy-handle = <&rgmii_phy0>;
    status = "disbaled";
};
```

`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-port.dtsi`
```
&gmac1 {                                                                                                                                                                
    /* Use rgmii-rxid mode to disable rx delay inside Soc */
    phy-mode = "rgmii-rxid";
    clock_in_out = "output";

    snps,reset-gpio = <&gpio3 RK_PB7 GPIO_ACTIVE_LOW>;
    snps,reset-active-low;
    /* Reset time is 20ms, 100ms for rtl8211f */
    snps,reset-delays-us = <0 20000 100000>;

    pinctrl-names = "default";
    pinctrl-0 = <&gmac1_miim
            &gmac1_tx_bus2
            &gmac1_rx_bus2
            &gmac1_rgmii_clk
            &gmac1_rgmii_bus>;

    tx_delay = <0x42>;
    //rx_delay = <0x4f>;

    phy-handle = <&rgmii_phy1>;
    status = "disbaled";
};

```

#### 板级的配置
`kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588l.dtsi`
```
/* gamc0 */
&gmac0 {
    snps,reset-gpio = <&gpio3 RK_PC7 GPIO_ACTIVE_LOW>;
    tx_delay = <0x39>;
    status = "okay";
};

```

#### 查看IP地址
* 双以太网口接入网络，可以通过调试串口或者adb来查看IP地址，比如

    ```
    ifconfig eth0
    eth0      Link encap:Ethernet  HWaddr 52:22:4b:e4:f8:3c  Driver rk_gmac-dwmac
          inet addr:168.168.104.210  Bcast:168.168.255.255  Mask:255.255.0.0
          inet6 addr: 240e:3b1:f175:9df0:ea6d:9993:33e6:202d/64 Scope: Global
          inet6 addr: 240e:3b1:f175:9df0:e0e8:b021:fc7a:dafc/64 Scope: Global
          inet6 addr: fe80::9cd8:e64d:b9c2:6f4/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:229 errors:0 dropped:0 overruns:0 frame:0
          TX packets:62 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:40526 TX bytes:6698
          Interrupt:70


    ```

#### 连通性测试
* eth0

    ```
    ping -I eth0 -c 10  www.baidu.com
    PING www.a.shifen.com (14.215.177.38) from 168.168.104.210 eth0: 56(84) bytes of data.
    64 bytes from 14.215.177.38: icmp_seq=1 ttl=55 time=9.45 ms
    64 bytes from 14.215.177.38: icmp_seq=2 ttl=55 time=9.36 ms
    64 bytes from 14.215.177.38: icmp_seq=3 ttl=55 time=10.0 ms
    64 bytes from 14.215.177.38: icmp_seq=4 ttl=55 time=11.3 ms
    64 bytes from 14.215.177.38: icmp_seq=5 ttl=55 time=41.7 ms
    64 bytes from 14.215.177.38: icmp_seq=6 ttl=55 time=9.73 ms
    64 bytes from 14.215.177.38: icmp_seq=7 ttl=55 time=9.28 ms
    64 bytes from 14.215.177.38: icmp_seq=8 ttl=55 time=9.26 ms
    64 bytes from 14.215.177.38: icmp_seq=9 ttl=55 time=9.50 ms
    64 bytes from 14.215.177.38: icmp_seq=10 ttl=55 time=12.9 ms

    --- www.a.shifen.com ping statistics ---
    10 packets transmitted, 10 received, 0% packet loss, time 9013ms
    rtt min/avg/max/mdev = 9.265/13.262/41.757/9.562 ms
    ```