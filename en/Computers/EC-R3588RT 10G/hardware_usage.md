# Hardware Function Usage
## Debug Serial

The EC-R3588RT_10G does not expose a debug serial port on the enclosure. You need to remove the top cover to access the debug serial port on the ROC-RK3588-RT carrier board, and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## Display Interface

The EC-R3588RT_10G provides 2 x HDMI2.1 display output interfaces and 1 x Display Port 1.4 display output interface (multiplexed with the Type-C OTG interface):

* HDMI: HDMI output interface, supports the HDMI2.1 protocol, and a single port supports up to 7680x4320@60Hz (8K) output.
* Display Port: DP output interface (software node `dp0`), supports up to 7680x4320@30Hz output.

The RK3588 has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller (for example, Port0 can be connected to display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1). The maximum output resolutions of each port are as follows:

* Port0 supports up to 7680x4320@60Hz
* Port1 supports up to 4096x2304@60Hz
* Port2 supports up to 4096x2304@60Hz
* Port3 supports up to 1920x1080@60Hz

Multi-screen note: when the RK3588 performs 8K output, it uses the resources of both Port0 and Port1 at the same time, and the display controller connected to Port1 will output abnormally. In multi-screen scenarios, please pay attention to the allocation between the display controllers and the Video Ports.

### Debugging methods

* Set the resolution in the system: on Android, adjust it in `Settings -> Display -> HDMI -> Resolution settings`.
* Get the edid of HDMI/Display Port (take HDMI as an example):

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* Get the resolutions supported by HDMI/Display Port (take HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* Get the connection status of HDMI/Display Port (take HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* Get the information of the Video Portx currently in use in the system (and the connected display controller):

```
cat /d/dri/0/summary
```

Generally, if you encounter a problem where HDMI/Display Port cannot display, you should first run the above commands to check whether the connection status, edid and resolution are correct.

## Ethernet

The EC-R3588RT_10G provides 3 RJ45 Ethernet ports: 2 Gigabit ports and 1 2.5G port. The 2.5G port supports jumbo frames, and features high bandwidth and low latency. Each port is recognized as an `ethX` device in the system (the number is subject to the actual system).

In addition, the EC-R3588RT_10G also provides 2 x 10G optical fiber ports through PCIe expansion, suitable for high-speed optical fiber access scenarios.

### View IP address and connectivity test

After the Ethernet ports are connected to the network, you can view the IP addresses through the debug serial port or adb:

```
ifconfig eth0
ifconfig eth1
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

On Linux, you can also use `ip addr` or `ifconfig` to check the port status and IP addresses.

## M.2 Interface

The EC-R3588RT_10G provides 2 M.2 expansion interfaces:

* 1 x PCIe2.0 (M.2 NVMe): for expanding NVMe solid state drives.
* 1 x PCIe2.0 (M.2 E-KEY): for expanding WiFi6 / BT5.0 wireless modules.

## Watchdog

### Introduction

The watchdog is actually a timer that starts counting once it is started. The system or software needs to communicate with the watchdog within the specified time (commonly known as feeding the dog) to reset the count, and so on repeatedly, so as to confirm that the system and software are running normally.

If the dog is not fed within the specified time, the watchdog times out, indicating that the system or application is stuck in a loop or dead. At this time, the watchdog will send a reset signal to reset the main control and get rid of the deadlock.

### DTS configuration

The DTS node of the RK3588 watchdog is defined in the `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi` file:

```
wdt: watchdog@feaf0000 {
    compatible = "snps,dw-wdt";
    reg = <0x0 0xfeaf0000 0x0 0x100>;
    clocks = <&cru TCLK_WDT0>, <&cru PCLK_WDT0>;
    clock-names = "tclk", "pclk";
    interrupts = <GIC_SPI 315 IRQ_TYPE_LEVEL_HIGH>;
    status = "disabled";
};
```

The watchdog is disabled by default. You need to enable the wdt node in the DTS file before using it:

```
&wdt{
    status = "okay";
};
```

### Usage

The driver file of the watchdog is `kernel-5.10/drivers/watchdog/dw_wdt.c`. The internal watchdog device is named `/dev/watchdog`, and users can control it with the `echo` command:

```
# Write any content (except the capital letter 'V') to enable the watchdog; it needs to be written once every 44 seconds (feeding the dog)
echo A > /dev/watchdog

# Enable the watchdog, and the kernel will automatically feed the dog every 22 seconds
echo V > /dev/watchdog
```

You can also control the watchdog through a program: after opening `/dev/watchdog` with `open`, the watchdog starts counting immediately; use `ioctl` (`WDIOC_SETTIMEOUT`/`WDIOC_GETTIMEOUT`) to set and get the timeout, and call `write` in a loop to feed the dog. Note: when the user does not set the timeout, the driver applies the default requested timeout of 30s.

Reference document: SDK/RKDocs (docs for Linux)/common/watchdog

* [Device Tree Manual](linux_dts_manual.md)
