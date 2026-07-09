## Ethernet

### DTS configure

#### Common
`kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-common.dtsi`
```
&gmac0 {
        /* Use rgmii-rxid mode to disable rx delay inside Soc */
        phy-mode = "rgmii-rxid";
        clock_in_out = "output";

        snps,reset-gpio = <&gpio2 RK_PB5 GPIO_ACTIVE_LOW>;
        snps,reset-active-low;
        /* Reset time is 20ms, 100ms for rtl8211f */
        snps,reset-delays-us = <0 20000 100000>;

        pinctrl-names = "default";
        pinctrl-0 = <&eth0m0_miim
                     &eth0m0_tx_bus2
                     &eth0m0_rx_bus2
                     &eth0m0_rgmii_clk
                     &eth0m0_rgmii_bus
                     &ethm0_clk0_25m_out>;

        tx_delay = <0x21>;
        /* rx_delay = <0x3f>; */

        phy-handle = <&rgmii_phy0>;
        status = "okay";
};

&mdio0 {
        status = "okay";
        rgmii_phy0: phy@1 {
                compatible = "ethernet-phy-ieee802.3-c22";
                reg = <0x1>;
                clocks = <&cru REFCLKO25M_GMAC0_OUT>;
        };
};
```

### How to use dual Ethernet
Android The dual Ethernet port is divided into internal network and external network.

<font color=#FF0000>Note: The eth1 secondary network interface is a USB 2.0 extended network interface, so the maximum speed is 100M</font>

| hardware device name |driver | Android system device name | Primary and auxiliary |speed|
|---|---|---|---|---|
| eth0 | rk_gmac | Ethernet | Primary network port for external network| 1000Mpbs|
| eth1 | r8152 | Ethernet 2 | Auxiliary network port for intranet|100Mpbs|

![](../../../rk3576_img/EC-R3576PC/usage_ethernet_interface.jpg)

#### IP Addrs
* get from debug or adb by ifconfig

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
	```
	ifconfig eth1
	eth1      Link encap:Ethernet  HWaddr e2:76:ef:f2:13:4f  Driver r8152
          inet addr:168.168.105.2  Bcast:168.168.255.255  Mask:255.255.0.0
          inet6 addr: 240e:3b1:f175:9df0:7355:7ec7:fed3:ecab/64 Scope: Global
          inet6 addr: 240e:3b1:f175:9df0:2e23:792d:f04a:2975/64 Scope: Global
          inet6 addr: fe80::564f:1114:e491:4bbd/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:124 errors:0 dropped:1 overruns:0 frame:0
          TX packets:18 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:12387 TX bytes:2072


	```

#### PING Test
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

* eth1

	```
	ping -I eth1 -c 10 168.168.4.168
	PING 168.168.4.168 (168.168.4.168): 56 data bytes
	64 bytes from 168.168.4.168: seq=0 ttl=64 time=2.408 ms
	64 bytes from 168.168.4.168: seq=1 ttl=64 time=1.574 ms
	64 bytes from 168.168.4.168: seq=2 ttl=64 time=1.669 ms
	64 bytes from 168.168.4.168: seq=3 ttl=64 time=1.913 ms
	64 bytes from 168.168.4.168: seq=4 ttl=64 time=1.666 ms
	64 bytes from 168.168.4.168: seq=5 ttl=64 time=1.500 ms
	64 bytes from 168.168.4.168: seq=6 ttl=64 time=1.631 ms
	64 bytes from 168.168.4.168: seq=7 ttl=64 time=1.740 ms
	64 bytes from 168.168.4.168: seq=8 ttl=64 time=1.590 ms
	64 bytes from 168.168.4.168: seq=9 ttl=64 time=1.924 ms

	--- 168.168.4.168 ping statistics ---	
	10 packets transmitted, 10 packets received, 0% packet loss
	round-trip min/avg/max = 1.500/1.761/2.408 ms

	```