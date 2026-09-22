# Hardware Function Usage
## Debug Serial

The EC-A3588Q does not expose a debug serial port on the enclosure. You need to open the host before you can access the debug serial port on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The EC-A3588Q provides 1 CAN interface. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

The CAN device is recognized as `can0` in the system. The communication test commands are as follows:

```
# Shut down the can0 device
ip link set can0 down
# Set the bitrate to 250Kbps
ip link set can0 type can bitrate 250000
# Bring up the can0 device
ip link set can0 up
# Run candump on the receiving end, blocking and waiting for frames
candump can0
# Run cansend on the sending end to send a frame
cansend can0 123#1122334455667788
```

The `candump` and `cansend` tools are included in the SDK, and can also be obtained from can-utils.

Debugging and verification notes:

* The bitrates of both communication ends must be configured to be the same, otherwise no frames can be received; you can use `ip -details link show can0` to view the detailed configuration and status of the CAN device.
* If no frames are received after sending, please check whether the bus CAN_H and CAN_L are loose or reversed.

## UART (RS232 / RS485)

The EC-A3588Q provides 2 RS232 interfaces (RS232_0, RS232_1) and 1 RS485 interface. RS232_0 is expanded from the main control UART0, RS232_1 is expanded from the main control UART5, and RS485 is expanded from the main control UART1. It is recommended to use the official FC10 to DB9 serial cable. The pin order of serial cables from different manufacturers may be different, which will cause the serial port communication to fail.

The software nodes corresponding to each hardware interface:

```
RS232_0:  /dev/ttyS0
RS232_1:  /dev/ttyS5
RS485:    /dev/ttyS1
```

### Send and receive verification

Take RS485 (`/dev/ttyS1`) as an example, use the USB to serial adapter of the host computer to connect to the development board (the adapter node on the host side is subject to the actual one, such as `/dev/ttyUSB0`).

The development board sends, and the host receives:

```
# Run on the host terminal first
cat /dev/ttyUSB0
# Run on the debug serial terminal of the development board
echo "firefly RS485 test..." > /dev/ttyS1
```

The host terminal will receive the string `firefly RS485 test...`.

The host sends, and the development board receives:

```
# Run on the debug serial terminal of the development board first
busybox stty -echo -F /dev/ttyS1       # Turn off echo
cat /dev/ttyS1
# Run on the host terminal
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The debug serial terminal of the development board will receive the string `firefly RS485 test...`. The verification methods of RS232_0 and RS232_1 are the same; just replace the nodes with the corresponding `/dev/ttyS0` and `/dev/ttyS5`.

## SIM Card

The SIM card slot of the EC-A3588Q is used together with a 4G/5G module to provide mobile network connectivity. **The 4G/5G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3588_img/EC-A3588Q/sim_insert_direction.png" width="400">
</center>

## Display

The EC-A3588Q provides multiple display output interfaces and one video input interface:

* HDMI2.1: HDMI output interface, supports the HDMI2.1 protocol, up to 7680x4320@60Hz (8K) output.
* USB-C (DP1.4): Display Port output interface (software node `dp0`), up to 7680x4320@30Hz output.
* VGA: implemented by a DP to VGA conversion chip (software node `dp1`), up to 1080p@60Hz output.
* HDMI-IN: HDMI input interface, supports the HDMI2.0 protocol, up to 4K@60fps input.

The RK3588 has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller (for example, Port0 can be used to connect to display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1). The maximum resolutions that each port can output are as follows:

* Port0 can output up to 7680x4320@60Hz
* Port1 can output up to 4096x2304@60Hz
* Port2 can output up to 4096x2304@60Hz
* Port3 can output up to 1920x1080@60Hz

Notes on multi-screen usage: when the RK3588 does 8K output, it occupies the resources of both Port0 and Port1 at the same time. The default configuration of the SDK is HDMI0 (`hdmi0_in_vp0`) + DP0 (`dp0_in_vp2`) + HDMI-IN. With this configuration, HDMI2.1 (8K) and USB-C (DP1.4, 4K@60Hz) can output normally at the same time; do not reconfigure DP0 to vp1, otherwise the DP0 display will be abnormal when HDMI0 outputs 8K.

Usage of HDMI-IN: the Android system comes with two APKs, **Live Tv** and **RockchipCamera2**. Open them to display the input picture of HDMI-IN; the Ubuntu firmware integrates a test script, just run `/usr/local/bin/test_hdmirx.sh` to preview the input.

### Debugging methods

* Set the resolution in the system: on Android, adjust the resolution in `Settings -> Display -> HDMI -> Resolution settings`.
* Get the edid of the HDMI/Display Port (taking HDMI as an example):

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* Get the resolutions supported by the HDMI/Display Port (taking HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* Get the connection status of the HDMI/Display Port (taking HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* Get the information of the Video Portx currently in use in the system (and the connected display controller):

```
cat /d/dri/0/summary
```

Generally, if you encounter the problem that the HDMI/Display Port cannot display, you need to run the above commands first to check whether the connection status, edid and resolution are correct.

## Ethernet

The EC-A3588Q provides 2 RJ45 Gigabit network ports, corresponding to the `eth0` and `eth1` devices in the system.

### Using dual Ethernet

For the Android dual Ethernet ports, one is for the external network and the other is for the internal network. The mapping between the primary and secondary ports is as follows:

| Hardware device name | dts node | Android system device name | Primary/secondary relationship |
|---|---|---|---|
| eth1 | gmac0 | Ethernet | Primary port, used for the external network |
| eth0 | gmac1 | Ethernet 2 | Secondary port, used for the internal network |

### Viewing IP addresses and connectivity test

After both network ports are connected to the network, you can view the IP addresses through the debug serial port or adb:

```
ifconfig eth0
ifconfig eth1
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

When testing the internal network port (`eth0`), please modify the target IP according to the actual internal network environment. Under Linux, you can use `ip addr` or `ifconfig` to view the status and IP address of the network ports.

## RTC

The core board iCore-3588Q carried by the EC-A3588Q uses the HYM8563 as the RTC (Real Time Clock). The HYM8563 is a low-power CMOS real-time clock/calendar chip. All addresses and data are transferred serially through the I2C bus interface. It can count seconds, minutes, hours, weeks, days, months and years based on a 32.768kHz crystal. Driver reference: `kernel-5.10/drivers/rtc/rtc-hym8563.c`.

Linux provides three user-space calling interfaces, and the corresponding paths are (the device number is subject to the actual system):

* SYSFS interface: /sys/class/rtc/rtc0/
* PROCFS interface: /proc/driver/rtc
* IOCTL interface: /dev/rtc0

### SYSFS interface

You can directly use `cat` and `echo` to operate the interfaces under `/sys/class/rtc/rtc0/`.

For example, check the current RTC date and time:

```
# cat /sys/class/rtc/rtc0/date
2022-06-21
# cat /sys/class/rtc/rtc0/time
06:52:08
```

Set the power-on time, for example, power on after 120 seconds:

```
# Timed power-on after 120 seconds
echo +120 > /sys/class/rtc/rtc0/wakealarm
# Check the power-on time
cat /sys/class/rtc/rtc0/wakealarm
# Shut down
reboot -p
```

### PROCFS interface

Print the RTC-related information:

```
# cat /proc/driver/rtc
rtc_time        : 06:53:50
rtc_date        : 2022-06-21
alrm_time       : 06:55:05
alrm_date       : 2022-06-21
alarm_IRQ       : yes
alrm_pending    : no
update IRQ enabled      : no
periodic IRQ enabled    : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

### IOCTL interface

You can use `ioctl` to control `/dev/rtc0`. For detailed usage, please refer to the document `kernel-5.10/Documentation/admin-guide/rtc.rst`.

### FAQs

#### Q: The time is not synchronized after power-on?

A: Check whether the RTC battery is properly connected (the RTC power supply configuration of the computer is subject to the actual product).

## Watchdog

### Introduction

The watchdog is actually a timer. After it is started, it begins to count. The system or software needs to communicate with the watchdog within the specified time (commonly known as feeding the dog) to reset the count, and so on repeatedly, so as to confirm that the system and software are running normally.

If the dog is not fed within the specified time, the watchdog will time out, indicating that the system or application has fallen into a loop or is stuck. At this time, the watchdog will send a reset signal to reset the main control and get rid of the deadlock.

### DTS Configuration

The DTS node of the RK3588 watchdog is defined in the file `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi`:

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

The driver file of the watchdog is `kernel-5.10/drivers/watchdog/dw_wdt.c`. The device name of the internal watchdog is `/dev/watchdog`, and users can control the device with the `echo` command:

```
# Write any content (except the uppercase letter 'V') to enable the watchdog. It needs to be written once within every 44 seconds (feeding the dog)
echo A > /dev/watchdog

# Enable the watchdog, and the kernel will automatically feed the dog every 22 seconds
echo V > /dev/watchdog
```

You can also control the watchdog through a program: after opening `/dev/watchdog` with `open`, the watchdog starts counting immediately. Use `ioctl` (`WDIOC_SETTIMEOUT`/`WDIOC_GETTIMEOUT`) to set and get the timeout, and loop `write` to feed the dog. Note that: when the user has not set the timeout, the driver will apply the default requested timeout of 30s; the driver maintains a preset timeout list internally, and will find a suitable time in the list as the final set timeout, so the final effective timeout is not necessarily equal to the time passed in by the application layer.

Reference document: SDK/RKDocs (docs for Linux)/common/watchdog

If the computer has an external hardware watchdog, a device file in the form of `wdt_XXX` will be generated under `/dev/` (which can be confirmed by `ls /dev/wdt_*`). Enabling and feeding the dog can be done by writing to the device file (whether it is supported is subject to the actual system).
