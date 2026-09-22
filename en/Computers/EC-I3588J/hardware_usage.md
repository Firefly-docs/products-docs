# Hardware Function Usage
## Debug Serial

The ITX-3588J  does not expose a debug serial port on the enclosure. You need to open the chassis before you can access the debug serial port on the mainboard, and connect an external USB to TTL serial module for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The ITX-3588J  provides 1 CAN interface (added in V1.1 version; the H and L markings are reversed in V1.1, and the silkscreen is correct for V1.2 and subsequent versions. Please check the terminal silkscreen before wiring). When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

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

The ITX-3588J  provides 1 RS232 interface and 1 RS485 interface. RS232 is expanded from the main control UART0 (shared with UART0 through jumpers, select one of them), and RS485 is expanded from the main control UART1 (shared with UART1 through jumpers, select one of them). The DTS configuration is located in `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dtsi`. It is recommended to use the official FC10 to DB9 serial cable for RS232 and RS485. The pin order of serial cables from different manufacturers may be different, which will cause the serial port communication to fail.

The software nodes corresponding to each hardware interface (subject to the actual system):

```
RS232:  /dev/ttyS0
RS485:  /dev/ttyS1
```

### Send and receive verification

Take RS485 (`/dev/ttyS1`) as an example, use the USB to serial adapter of the host computer to connect to the device (the adapter node on the host side is subject to the actual one, such as `/dev/ttyUSB0`).

The device sends, and the host receives:

```
# Run on the host terminal first
cat /dev/ttyUSB0
# Run on the debug serial terminal of the device
echo "firefly RS485 test..." > /dev/ttyS1
```

The host terminal will receive the string `firefly RS485 test...`.

The host sends, and the device receives:

```
# Run on the debug serial terminal of the device first
busybox stty -echo -F /dev/ttyS1       # Turn off echo
cat /dev/ttyS1
# Run on the host terminal
echo "firefly RS485 test..." > /dev/ttyUSB0
```

The debug serial terminal of the device will receive the string `firefly RS485 test...`. The verification method of RS232 is the same; just replace the node with the corresponding `/dev/ttyS0`.

## Display

The ITX-3588J  provides multiple display output interfaces, one video input interface and 2 MIPI DSI display interfaces:

* HDMI0: HDMI output interface, supports the HDMI2.1 protocol, up to 7680x4320@60Hz (8K) output.
* HDMI1: HDMI output interface, supports the HDMI2.0 protocol, up to 4096x2304@60Hz output.
* Display Port1.4: DP output interface, up to 7680x4320@30Hz (8K) output.
* VGA: implemented by a DP to VGA conversion chip (software node `dp1`), limited by the VGA protocol, up to 1080p@60Hz output.
* HDMI-IN: HDMI input interface, supports the standard HDMI2.0 protocol, up to 4K@60fps input.
* 2 x MIPI DSI: MIPI DSI display output interfaces.

The RK3588 has four Video output ports (Port0 ~ Port3), and each Video output port is bound to a fixed display controller (for example, Port0 can be used to connect to display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1). The maximum resolutions that each port can output are as follows:

* Port0 can output up to 7680x4320@60Hz
* Port1 can output up to 4096x2304@60Hz
* Port2 can output up to 4096x2304@60Hz
* Port3 can output up to 1920x1080@60Hz

Notes on multi-screen usage: when the RK3588 does 8K output, it occupies the resources of both Port0 and Port1 at the same time, and the output of the display controller connected to Port1 will be abnormal; only when Port0 outputs less than or equal to 4K@60Hz can Port1 output 4K@60Hz normally. In addition, currently one Portx can only output one resolution format at the same time, please allocate the display controllers and Video Ports reasonably.

### Usage of HDMI-IN

The Android system comes with two APKs, **Live Tv** and **RockchipCamera2**. Open them to display the input picture of HDMI-IN; the Ubuntu firmware integrates the `test_hdmirx.sh` test script, just run `/usr/local/bin/test_hdmirx.sh` to preview the input.

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

The ITX-3588J  provides 2 RJ45 network ports (support 1Gbps), corresponding to the `eth0` and `eth1` devices in the system (the device numbers are subject to the actual system).

After the network ports are connected to the network, you can view the IP addresses through the debug serial port or adb:

```
ifconfig eth0
ifconfig eth1
```

Connectivity test:

```
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

## USB Interfaces

The USB interfaces of the ITX-3588J  include:

* 4 x USB3.0: used to connect USB peripherals.
* 4 x USB2.0: used to connect USB peripherals.
* 1 x USB3.0 OTG (Type-C): OTG/download interface, mainly used for firmware flashing, and can also be used as OTG.

## Storage Interfaces

The ITX-3588J  provides rich storage expansion interfaces:

* 4 x SATA3.0: can connect SATA hard disks/solid state drives.
* 1 x M.2 (SATA3.0): can connect M.2 SATA protocol solid state drives.
* 1 x PCIe3.0x4: can expand NVMe protocol solid state drives and other PCIe devices.
* 1 x TF Card: TF card slot. Please power off the device before inserting or removing the card. The card insertion direction is subject to the silkscreen of the card slot.

Take the SATA device as an example, the device is recognized as `sdX` in the system (subject to the actually recognized device), manual mounting:

```
# Format to the EXT4 file system
mkfs.ext4 /dev/sda
# Mount
mount /dev/sda /mnt/
# Check the mounting path
df -h
```

## SIM Card

The SIM card slot of the ITX-3588J  is used together with a 4G/5G module to provide mobile network connectivity; WIFI (supporting WiFi6) and Bluetooth are provided by on-board/external modules. **The 4G/5G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../rk3588_img/EC-I3588J/sim_insert_direction.png" width="400">
</center>

## Audio Interfaces

The ITX-3588J  provides Speaker and Line-In interfaces, which are used to connect sound output devices such as passive speakers and external audio sources respectively.

* [Device Tree Manual](linux_dts_manual.md)
