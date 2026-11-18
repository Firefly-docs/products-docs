# Ethernet Usage

## Introduction [step]

Firefly-R3572A provides 2 RJ45 Gigabit Ethernet ports (1000 Mbps), both with automatic MDI/MDIX crossover.

## DTS Configuration [step]

The GMAC controller nodes are defined in the chip-level DTS of the SDK. The board-level configuration includes the PHY reset pin, the clock delays (tx_delay/rx_delay) and the pinctrl pin multiplexing. A typical configuration structure is as follows (the actual properties and pins depend on the SDK code):

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

The two ports correspond to two GMAC controller nodes. The pin multiplexing and delay parameters need to be adjusted according to the actual hardware measurement.

## Checking the Network Interfaces [step]

Check the network interface status after the system boots:

```bash
ifconfig -a
```

The 2 ports are named `eth0` and `eth1` by default; the naming order depends on the system enumeration. Check the negotiated rate:

```bash
ethtool eth0
```

`Speed` showing `1000Mb/s` means the port is working in Gigabit mode.

## Obtaining an IP Address [step]

The ports obtain an address through DHCP by default. A static address can also be configured:

```bash
udhcpc -i eth0                          # Obtain through DHCP
ifconfig eth0 192.168.1.100 netmask 255.255.255.0   # Static configuration
```

## Connectivity Test [step]

Test the connectivity of both ports:

```bash
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 www.baidu.com
```

With a normal network there is no packet loss, and the `rtt` time depends on the network environment.

## Hardware Connection [step]

The RJ45 connectors have integrated magnetics. Connect a router or a switch with a standard network cable. The connectors are located on the board edge; their positions are shown in the interface diagrams of [Introduction](../product_introduction/started.md).
