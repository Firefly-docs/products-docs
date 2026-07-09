## Ethernet 使用

### dts 配置

#### 公共的配置

`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b.dtsi`

```
gmac: ethernet@21c70000 {
        compatible = "rockchip,rv1126b-gmac", "snps,dwmac-4.20a";
        reg = <0x21c70000 0x10000>;
        interrupts = <GIC_SPI 200 IRQ_TYPE_LEVEL_HIGH>,
                     <GIC_SPI 197 IRQ_TYPE_LEVEL_HIGH>;
        interrupt-names = "macirq", "eth_wake_irq";
        rockchip,grf = <&grf>;
        rockchip,php_grf = <&ioc_grf>;
        clocks = <&cru CLK_GMAC_125M>, <&cru CLK_50M_GMAC_IOBUF_VI>,
                 <&cru PCLK_GMAC>, <&cru ACLK_GMAC>,
                 <&cru CLK_GMAC_PTP_REF>;
        clock-names = "stmmaceth", "clk_mac_ref",
                      "pclk_mac", "aclk_mac",
                      "ptp_ref";
        resets = <&cru SRST_ARESETN_GMAC>;
        reset-names = "stmmaceth";

        snps,mixed-burst;
        snps,tso;
        ...
```

#### 板级的配置

`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bq38.dtsi`

```
#ifdef FF_GMAC
&gmac {
        phy-mode = "rgmii";
        clock_in_out = "output";

        snps,reset-gpio = <&gpio5 RK_PB4 GPIO_ACTIVE_LOW>;
        snps,reset-active-low;
        /* Reset time is 20ms, 100ms for rtl8211f */
        snps,reset-delays-us = <0 20000 100000>;

        assigned-clocks = <&cru CLK_MAC_OUT2IO>;
        assigned-clock-rates = <25000000>;

        pinctrl-names = "default";
        pinctrl-0 = <&ethm1_miim_pins
                     &ethm1_tx_bus2_pins
                     &ethm1_rx_bus2_pins
                     &ethm1_rgmii_clk_pins
                     &ethm1_rgmii_bus_pins
                     &eth_clk_25mm1_out_pins>;

        tx_delay = <0x29>;
        rx_delay = <0x16>;

        phy-handle = <&rgmii_phy>;
        status = "okay";
};

&mdio {
        rgmii_phy: rgmii-phy@0 {
                compatible = "ethernet-phy-ieee802.3-c22";
                reg = <0x0>;
                clocks = <&cru CLK_MAC_OUT2IO>;
        };
};
#endif
```

#### 查看IP地址
* 双以太网口接入网络，可以通过调试串口或者adb来查看IP地址，比如

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

#### 连通性测试
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