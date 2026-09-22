# Hardware Function Usage
## Debug Serial

The Station-M3 does not expose a debug serial port on the enclosure. You need to remove the top cover to access the debug serial port on the carrier board (ROC-RK3588S-PC) and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## Display Interface

The Station-M3 provides one HDMI2.1 interface (up to 7680x4320@60Hz output), one Display Port 1.4 interface (up to 7680x4320@30Hz output, software node `dp0`) and two MIPI DSI display output interfaces, supporting multi-screen display.

The RK3588S has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller. The maximum resolution of each port is as follows:

* Port0 supports up to 7680x4320@60Hz
* Port1 supports up to 4096x2304@60Hz
* Port2 supports up to 4096x2304@60Hz
* Port3 supports up to 1920x1080@60Hz

Note for multi-screen usage: when the RK3588S outputs 8K, it occupies the resources of both Port0 and Port1 at the same time, and the display controller connected to Port1 will work abnormally; only when Port0 outputs at or below 4K@60Hz can Port1 output 4K@60Hz normally.

For the software configuration of the display interfaces, refer to: `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`.

### Debugging

* Get the edid of HDMI/Display Port (taking HDMI as an example):

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
```

* Get the resolutions supported by HDMI/Display Port (taking HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/modes
```

* Get the connection status of HDMI/Display Port (taking HDMI as an example):

```
cat /sys/class/drm/card0-HDMI-A-1/status
```

* Get the information of the Video Portx in use (and the connected display controller) in the system:

```
cat /d/dri/0/summary
```

Generally, if HDMI/Display Port cannot display normally, you need to run the above commands first to check whether the connection status, edid and resolution are correct.

## Ethernet

The Station-M3 provides one RJ45 port (1Gbps), corresponding to the `eth0` device in the system.

After connecting to the network, you can view the IP address through the debug serial port or adb:

```
ifconfig eth0
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
```

## SATA

The Station-M3 provides one M.2 interface, which is configured as M.2 SATA3.0 by default (supporting SSDs with the SATA protocol), and can also be configured as M.2 PCIe2.0 by software (supporting SSDs with the NVMe protocol). The system needs to be restarted after the configuration is changed.

The DTS configuration is located in `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`:

```
#define M2_SATA_OR_PCIE 1 /*1 = SATA , 0 = PCIe */

/* default use sata3.0 , pcie2.0 optional*/
&combphy0_ps {
    status = "okay";
};

#if M2_SATA_OR_PCIE
&sata0 {
    pinctrl-names = "default";
    pinctrl-0 = <&sata_reset>;
    status = "okay";
};
#else
&pcie2x1l2 {
    reset-gpios = <&gpio3 RK_PD1 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie20>;
    status = "okay";
};
#endif
```

Taking SATA3.0 as an example, the device is recognized as `sda` in the system (`/dev/block/sda` on Android). Manual mounting:

```
# Format it as an EXT4 file system
mkfs.ext4 /dev/sda
# Mount it
mount /dev/sda /mnt/
# Check the mounting path
df -h
```

## RTC

The Station-M3 uses the HYM8563 as the RTC (Real Time Clock). The HYM8563 is a low-power CMOS real-time clock/calendar chip. All addresses and data are transferred serially via the I2C bus interface, and it can count seconds, minutes, hours, weeks, days, months and years based on a 32.768kHz crystal. Driver reference: `kernel-5.10/drivers/rtc/rtc-hym8563.c`.

Linux provides three user-space call interfaces, and the corresponding paths are (the device number is subject to the actual system):

* SYSFS interface: /sys/class/rtc/rtc0/
* PROCFS interface: /proc/driver/rtc
* IOCTL interface: /dev/rtc0

For example, check the current date and time of the RTC:

```
# cat /sys/class/rtc/rtc0/date
2022-06-21
# cat /sys/class/rtc/rtc0/time
06:52:08
```

Set the power-on time, for example, power on after 120 seconds:

```
# Scheduled power-on after 120 seconds
echo +120 > /sys/class/rtc/rtc0/wakealarm
# Check the power-on time
cat /sys/class/rtc/rtc0/wakealarm
# Power off
reboot -p
```

## Watchdog

The watchdog is actually a timer. After it is started, it begins to count. The system or software needs to communicate with the watchdog (commonly known as feeding the dog) within the specified time to reset the count. If the watchdog is not fed within the specified time, it will time out and send a reset signal to reset the main controller, so that the system gets rid of the deadlock.

The DTS node of the RK3588S watchdog is defined in `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi`. The watchdog is disabled by default, and the wdt node needs to be enabled in the DTS file before it can be used:

```
&wdt{
    status = "okay";
};
```

The device name of the internal watchdog is `/dev/watchdog`. Users can control it with the `echo` command:

```
# Write any content (except the capital letter 'V') to enable the watchdog; it needs to be fed (written) once every 44 seconds
echo A > /dev/watchdog

# Enable the watchdog, and the kernel will feed the dog automatically every 22 seconds
echo V > /dev/watchdog
```

* [Device Tree Manual](linux_dts_manual.md)
