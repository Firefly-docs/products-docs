## Ethernet

### DTS configure

#### Common
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
`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dtsi`
```
/* gmac1 */
&gmac1{
 	snps,reset-gpio = <&gpio1 RK_PA1 GPIO_ACTIVE_LOW>;
	tx_delay = <0x37>;
	status = "okay";
};

```

#### IP Addrs
* get from debug or adb by ifconfig

	```
	ifconfig eth0
	eth0      Link encap:Ethernet  HWaddr 02:df:a5:01:03:cf  Driver rk_gmac-dwmac
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 TX bytes:0
          Interrupt:80
	```

#### PING Test
* eth0

	```
	rk3588s_firefly_aio_3588sg:/ # ping -I eth0 -c 10  www.baidu.com
    PING www.a.shifen.com (14.215.177.38) from 168.168.103.73 eth0: 56(84) bytes of data.
    64 bytes from 14.215.177.38: icmp_seq=1 ttl=56 time=12.2 ms
    64 bytes from 14.215.177.38: icmp_seq=2 ttl=56 time=11.9 ms
    64 bytes from 14.215.177.38: icmp_seq=3 ttl=56 time=11.9 ms
    64 bytes from 14.215.177.38: icmp_seq=4 ttl=56 time=11.9 ms
    64 bytes from 14.215.177.38: icmp_seq=5 ttl=56 time=12.0 ms
    64 bytes from 14.215.177.38: icmp_seq=6 ttl=56 time=13.3 ms
    64 bytes from 14.215.177.38: icmp_seq=7 ttl=56 time=11.4 ms
    64 bytes from 14.215.177.38: icmp_seq=8 ttl=56 time=12.0 ms
    64 bytes from 14.215.177.38: icmp_seq=9 ttl=56 time=12.4 ms
    64 bytes from 14.215.177.38: icmp_seq=10 ttl=56 time=11.8 ms

    --- www.a.shifen.com ping statistics ---
    10 packets transmitted, 10 received, 0% packet loss, time 9015ms
    rtt min/avg/max/mdev = 11.447/12.148/13.309/0.462 ms
	```
