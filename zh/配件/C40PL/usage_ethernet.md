## Ethernet 使用

### dts 配置

#### 公共的配置
`kernel/arch/arm/boot/dts/rv1126-firefly-rk809.dtsi`

```
&gmac {
        phy-mode = "rgmii";
        clock_in_out = "output";

        snps,reset-gpio = <&gpio2 RK_PB2 GPIO_ACTIVE_LOW>;
        snps,reset-active-low;
        /* Reset time is 20ms, 100ms for rtl8211f */
        snps,reset-delays-us = <0 20000 100000>;

        assigned-clocks = <&cru CLK_GMAC_SRC>, <&cru CLK_GMAC_TX_RX>, <&cru CLK_GMAC_ETHERNET_OUT>;
        assigned-clock-parents = <&cru CLK_GMAC_SRC_M1>, <&cru RGMII_MODE_CLK>;//核心板和底板使用这个
        //assigned-clock-parents = <&cru CLK_GMAC_SRC_M0>, <&cru RGMII_MODE_CLK>;//集群服务器R1使用这个
        assigned-clock-rates = <125000000>, <0>, <25000000>;

        pinctrl-names = "default";
        pinctrl-0 = <&rgmiim1_miim &rgmiim1_bus2 &rgmiim1_bus4 &clkm1_out_ethernet>;
        //pinctrl-0 = <&rgmiim0_pins &clk_out_ethernetm0_pins>;//集群服务器R1使用这个

        tx_delay = <0x2a>;
        rx_delay = <0x1a>;

        phy-handle = <&phy>;
        status = "okay";
};
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