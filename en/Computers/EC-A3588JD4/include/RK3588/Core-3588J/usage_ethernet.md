## Ethernet

### DTS configure

#### Common
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

#### Board
`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dtsi`
```
/* gamc0 */
&gmac0 {                                                                                                                                                                
    snps,reset-gpio = <&gpio3 RK_PC7 GPIO_ACTIVE_LOW>;
    tx_delay = <0x45>;
    status = "okay";
};

&gmac0_tx_bus2{
    rockchip,pins =
    /* gmac0_txd0 */
    <2 RK_PB6 1 &pcfg_pull_up_drv_level_6>,
    /* gmac0_txd1 */
    <2 RK_PB7 1 &pcfg_pull_up_drv_level_6>,
    /* gmac0_txen */
    <2 RK_PC0 1 &pcfg_pull_none>;
};

&gmac0_rgmii_bus{
    rockchip,pins =
    /* gmac0_rxd2 */
    <2 RK_PA6 1 &pcfg_pull_none>,
    /* gmac0_rxd3 */
    <2 RK_PA7 1 &pcfg_pull_none>,
    /* gmac0_txd2 */
    <2 RK_PB1 1 &pcfg_pull_up_drv_level_6>,
    /* gmac0_txd3 */
    <2 RK_PB2 1 &pcfg_pull_up_drv_level_6>;
};

/* gmac1 */
&gmac1 {
    snps,reset-gpio = <&gpio3 RK_PB7 GPIO_ACTIVE_LOW>;
    tx_delay = <0x42>;
    status = "okay";
};

&gmac1_tx_bus2{
    rockchip,pins =
    /* gmac1_txd0 */
    <3 RK_PB3 1 &pcfg_pull_up_drv_level_6>,
    /* gmac1_txd1 */
    <3 RK_PB4 1 &pcfg_pull_up_drv_level_6>,
    /* gmac1_txen */
    <3 RK_PB5 1 &pcfg_pull_none>;
};

&gmac1_rgmii_bus{
    rockchip,pins =
    /* gmac1_rxd2 */
    <3 RK_PA2 1 &pcfg_pull_none>,
    /* gmac1_rxd3 */
    <3 RK_PA3 1 &pcfg_pull_none>,
    /* gmac1_txd2 */
    <3 RK_PA0 1 &pcfg_pull_up_drv_level_6>,
    /* gmac1_txd3 */
    <3 RK_PA1 1 &pcfg_pull_up_drv_level_6>;
};

```

### How to use dual ethernet
Android dual ethernet had intranet and outer net.

| hardware device name | dts node | Android system device name | Primary and auxiliary |
|---|---|---|---|
| eth0 | gmac1 | Ethernet 2 | Auxiliary network port for intranet | 
| eth1 | gmac0 | Ethernet | Primary network port for external network |

![](../../../rk3588_img/EC-A3588JD4/usage_ethernet_interface.png)

#### IP Addrs
* get from debug or adb by ifconfig

	```
	ifconfig eth0
	eth0  Link encap:Ethernet  HWaddr 36:0a:d2:60:c9:05  Driver rk_gmac-dwmac
          inet addr:168.168.101.68  Bcast:168.168.255.255  Mask:255.255.0.0 
          inet6 addr: fe80::3290:2dc:3624:1dec/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:15804 errors:0 dropped:1 overruns:0 frame:0 
          TX packets:68 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:1000 
          RX bytes:1811566 TX bytes:6748 
          Interrupt:75 
	```
	```
	ifconfig eth1
	eth1  Link encap:Ethernet  HWaddr 32:0a:d2:60:c9:05  Driver rk_gmac-dwmac
          inet addr:168.168.101.38  Bcast:168.168.255.255  Mask:255.255.0.0 
          inet6 addr: 240e:3b1:f174:f00:d3fe:4d8e:fda9:3c0/64 Scope: Global
          inet6 addr: fe80::b556:9d3f:e684:bec4/64 Scope: Link
          inet6 addr: 240e:3b1:f174:f00:9d78:e75f:6167:15c1/64 Scope: Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:913 errors:0 dropped:1 overruns:0 frame:0 
          TX packets:210 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:1000 
          RX bytes:108473 TX bytes:24906 
          Interrupt:127 
	```

#### PING Test
* eth0

	```
	ping -I eth0 -c 10 168.168.100.149                                        
	PING 168.168.100.149 (168.168.100.149) from 168.168.101.68 eth0: 56(84) bytes of data.
	64 bytes from 168.168.100.149: icmp_seq=1 ttl=64 time=0.424 ms
	64 bytes from 168.168.100.149: icmp_seq=2 ttl=64 time=0.561 ms
	64 bytes from 168.168.100.149: icmp_seq=3 ttl=64 time=0.538 ms
	64 bytes from 168.168.100.149: icmp_seq=4 ttl=64 time=0.507 ms
	64 bytes from 168.168.100.149: icmp_seq=5 ttl=64 time=0.755 ms
	64 bytes from 168.168.100.149: icmp_seq=6 ttl=64 time=0.657 ms
	64 bytes from 168.168.100.149: icmp_seq=7 ttl=64 time=0.696 ms
	64 bytes from 168.168.100.149: icmp_seq=8 ttl=64 time=0.784 ms
	64 bytes from 168.168.100.149: icmp_seq=9 ttl=64 time=0.544 ms
	64 bytes from 168.168.100.149: icmp_seq=10 ttl=64 time=0.750 ms

	--- 168.168.100.149 ping statistics ---
	10 packets transmitted, 10 received, 0% packet loss, time 9058ms
	rtt min/avg/max/mdev = 0.424/0.621/0.784/0.119 ms
	```

* eth1

	```
	ping -I eth1 -c 10 www.baidu.com
	PING www.a.shifen.com (14.215.177.39) from 168.168.101.38 eth1: 56(84) bytes of data.
	64 bytes from 14.215.177.39: icmp_seq=1 ttl=55 time=8.81 ms
	64 bytes from 14.215.177.39: icmp_seq=2 ttl=55 time=8.40 ms
	64 bytes from 14.215.177.39: icmp_seq=3 ttl=55 time=8.79 ms
	64 bytes from 14.215.177.39: icmp_seq=4 ttl=55 time=8.79 ms
	64 bytes from 14.215.177.39: icmp_seq=5 ttl=55 time=8.90 ms
	64 bytes from 14.215.177.39: icmp_seq=6 ttl=55 time=12.0 ms
	64 bytes from 14.215.177.39: icmp_seq=7 ttl=55 time=8.75 ms
	64 bytes from 14.215.177.39: icmp_seq=8 ttl=55 time=14.4 ms
	64 bytes from 14.215.177.39: icmp_seq=9 ttl=55 time=8.79 ms
	64 bytes from 14.215.177.39: icmp_seq=10 ttl=55 time=9.98 ms

	--- www.a.shifen.com ping statistics ---
	10 packets transmitted, 10 received, 0% packet loss, time 9015ms
	rtt min/avg/max/mdev = 8.405/9.775/14.422/1.854 ms
	```