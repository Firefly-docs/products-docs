## Ethernet

### Dts Configuration

#### Common Configuration

`kernel/arch/arm/boot/dts/rk3506.dtsi`
```
gmac0: ethernet@ff4c8000 {
	compatible = "rockchip,rk3506-gmac", "snps,dwmac-4.20a";
	reg = <0xff4c8000 0x2000>;
	interrupts = <GIC_SPI 66 IRQ_TYPE_LEVEL_HIGH>,
		     <GIC_SPI 69 IRQ_TYPE_LEVEL_HIGH>;
	interrupt-names = "macirq", "eth_wake_irq";
	rockchip,grf = <&grf>;
	clocks = <&cru CLK_MAC0>, <&cru CLK_MAC0_PTP>,
		 <&cru PCLK_MAC0>, <&cru ACLK_MAC0>;
	clock-names = "stmmaceth", "ptp_ref",
		      "pclk_mac", "aclk_mac";
	resets = <&cru SRST_A_MAC0>;
	reset-names = "stmmaceth";

	snps,mixed-burst;
	snps,tso;

	snps,axi-config = <&gmac0_stmmac_axi_setup>;
	snps,mtl-rx-config = <&gmac0_mtl_rx_setup>;
	snps,mtl-tx-config = <&gmac0_mtl_tx_setup>;

	phy-mode = "rmii";
...
};

gmac1: ethernet@ff4d0000 {
	compatible = "rockchip,rk3506-gmac", "snps,dwmac-4.20a";
	reg = <0xff4d0000 0x2000>;
	interrupts = <GIC_SPI 70 IRQ_TYPE_LEVEL_HIGH>,
		     <GIC_SPI 73 IRQ_TYPE_LEVEL_HIGH>;
	interrupt-names = "macirq", "eth_wake_irq";
	rockchip,grf = <&grf>;
	clocks = <&cru CLK_MAC1>, <&cru CLK_MAC1_PTP>,
		 <&cru PCLK_MAC1>, <&cru ACLK_MAC1>;
	clock-names = "stmmaceth", "ptp_ref",
		      "pclk_mac", "aclk_mac";
	resets = <&cru SRST_A_MAC1>;
	reset-names = "stmmaceth";

	snps,mixed-burst;
	snps,tso;

	snps,axi-config = <&gmac1_stmmac_axi_setup>;
	snps,mtl-rx-config = <&gmac1_mtl_rx_setup>;
	snps,mtl-tx-config = <&gmac1_mtl_tx_setup>;

	phy-mode = "rmii";
...
};
```

#### Board level configuration
`kernel/arch/arm/boot/dts/rk3506b-firefly-common.dtsi`
```
/* gmac0 */
&gmac0 {
        phy-mode = "rmii";
        clock_in_out = "input";

        pinctrl-names = "default";
        pinctrl-0 = <&eth_rmii0_miim_pins
                     &eth_rmii0_tx_bus2_pins
                     &eth_rmii0_rx_bus2_pins
                     &eth_rmii0_clk_pins>;

        phy-handle = <&rmii_phy0>;
        status = "okay";
};
&mdio0 {
        rmii_phy0: phy@1 {
                compatible = "ethernet-phy-ieee802.3-c22";
                reg = <0x1>;
        };
};

/* gmac1 */
&gmac1 {
        phy-mode = "rmii";
        clock_in_out = "input";

        pinctrl-names = "default";
        pinctrl-0 = <&eth_rmii1_miim_pins
                     &eth_rmii1_tx_bus2_pins
                     &eth_rmii1_rx_bus2_pins
                     &eth_rmii1_clk_pins>;

        phy-handle = <&rmii_phy1>;
        status = "okay";
};

&mdio1 {
        rmii_phy1: phy@1 {
                compatible = "ethernet-phy-ieee802.3-c22";
                reg = <0x1>;
        };
};
```

#### Check IP Address
* Dual Ethernet ports are connected to the network. You can check the IP address through the debug serial port or adb, for example
```
root@rk3506-buildroot:/# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr C6:A3:0A:E9:63:59
          inet addr:172.16.10.201  Bcast:172.16.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:131 errors:0 dropped:20 overruns:0 frame:0
          TX packets:5 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:16537 (16.1 KiB)  TX bytes:810 (810.0 B)
          Interrupt:49 Base address:0x6000

root@rk3506-buildroot:/# ifconfig eth1
eth1      Link encap:Ethernet  HWaddr CA:A3:0A:E9:63:59
          inet addr:172.16.10.203  Bcast:172.16.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:70 errors:0 dropped:12 overruns:0 frame:0
          TX packets:5 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:10235 (9.9 KiB)  TX bytes:810 (810.0 B)
          Interrupt:51 Base address:0x4000
```

#### Connectivity Test
* eth0
```
root@rk3506-buildroot:/# udhcpc -i eth0
udhcpc: started, v1.36.1
udhcpc: broadcasting discover
udhcpc: broadcasting select for 172.16.10.201, server 172.16.0.1
udhcpc: lease of 172.16.10.201 obtained from 172.16.0.1, lease time 7200
deleting routers
adding dns 202.96.128.166
adding dns 223.6.6.6

root@rk3506-buildroot:/# ping -I eth0 -c 10 www.baidu.com
PING www.baidu.com (183.2.172.42) from 172.16.10.201 eth0: 56(84) bytes of data.
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=1 ttl=54 time=10.1 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=2 ttl=54 time=12.9 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=3 ttl=54 time=10.6 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=4 ttl=54 time=12.2 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=5 ttl=54 time=6.69 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=6 ttl=54 time=7.72 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=7 ttl=54 time=6.85 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=8 ttl=54 time=7.76 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=9 ttl=54 time=8.37 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=10 ttl=54 time=7.58 ms

--- www.baidu.com ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 6.688/9.076/12.941/2.124 ms
```


* eth1
```
root@rk3506-buildroot:/# udhcpc -i eth1
udhcpc: started, v1.36.1
udhcpc: broadcasting discover
udhcpc: broadcasting select for 172.16.10.203, server 172.16.0.1
udhcpc: lease of 172.16.10.203 obtained from 172.16.0.1, lease time 7200
deleting routers
adding dns 202.96.128.166
adding dns 223.6.6.6

root@rk3506-buildroot:/# ping -I eth1 -c 10 www.baidu.com
PING www.baidu.com (183.2.172.42) from 172.16.10.203 eth1: 56(84) bytes of data.
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=1 ttl=54 time=17.2 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=2 ttl=54 time=6.93 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=3 ttl=54 time=11.1 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=4 ttl=54 time=8.54 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=5 ttl=54 time=12.0 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=6 ttl=54 time=12.6 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=7 ttl=54 time=7.93 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=8 ttl=54 time=7.17 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=9 ttl=54 time=7.92 ms
64 bytes from www.baidu.com (183.2.172.42): icmp_seq=10 ttl=54 time=11.4 ms

--- www.baidu.com ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 6.928/10.265/17.167/3.038 ms
```