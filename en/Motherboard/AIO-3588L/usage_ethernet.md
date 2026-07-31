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
`kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588l.dtsi`
```
/* gamc0 */
&gmac0 {
    snps,reset-gpio = <&gpio3 RK_PC7 GPIO_ACTIVE_LOW>;
    tx_delay = <0x39>;
    status = "okay";
};

```

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