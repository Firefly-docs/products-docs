# Hardware Function Usage
## Debug Serial

The EC-R3576PC does not expose a debug serial port on the enclosure. You need to open the host to access the debug serial port on the mainboard and connect an external USB to TTL serial module for debugging. For daily debugging, commands can also be executed through the ADB channel.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## CAN

The EC-R3576PC provides 1 CAN interface. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

<center>

<img alt="" src="../../../rk3576_img/EC-R3576PC/usage_can_interface.jpg" width="900">
</center>

The CAN device is recognized as `can0` in the system. The communication test commands are as follows (on Ubuntu, install the tools with `apt update && apt install can-utils`):

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

Debugging and verification notes:

* The bitrates of both communication ends must be configured consistently, otherwise no frame can be received; you can use `ip -details link show can0` to check the detailed configuration and status of the CAN device.
* If no frame is received after sending, check whether CAN_H and CAN_L on the bus are loose or reversed.

## UART (RS232 / RS485)

The EC-R3576PC provides 3 RS232 interfaces and 2 RS485 interfaces. The interface diagram is as follows:

<center>

<img alt="" src="../../../rk3576_img/EC-R3576PC/usage_uart_interface.jpg" width="900">
</center>

After the serial port is configured, the nodes corresponding to the hardware interfaces are as follows (see the interface diagram for the silkscreen):

```
RS485 :   /dev/ttysWK0(Silkscreen: A1 B1)    /dev/ttysWK1(Silkscreen: A2 B2)
RS232 :   /dev/ttysWK3(Silkscreen: T1 R1)    /dev/ttysWK2(Silkscreen: T2 R2)   /dev/ttyS6(Silkscreen: T3 R3)
```

The RS485 ports are expanded by an SPI to UART chip, and T3/R3 of the RS232 ports is expanded by the on-chip UART6.

### Send and receive verification

The simplest way is to short-circuit the TX/RX pins and then run commands in the debug serial terminal or ADB.

Taking `/dev/ttyS6` of RS232 as an example, after short-circuiting T3 and R3:

```
busybox stty -echo -F /dev/ttyS6          # disable echo
cat /dev/ttyS6 &                          # get the input string in the background
echo "firefly uart test..." > /dev/ttyS6  # send the string
```

The string `firefly uart test...` will then be received in the terminal.

For RS485 verification, short-circuit A1-A2 and B1-B2 (or connect to an external RS485 device), and replace the node in the commands above with `/dev/ttysWK0` and `/dev/ttysWK1`. The other RS232 nodes (`/dev/ttysWK2`, `/dev/ttysWK3`) work in the same way.

## Display Interface

The EC-R3576PC provides two display output interfaces, HDMI and Display Port, which support multi-screen duplicate/extended display:

* HDMI: supports the HDMI2.1 protocol with a maximum resolution of 4K@120Hz.
* Display Port: represented as `dp0` in the software, supports the DP TX 1.4a protocol with a maximum resolution of 4K@60Hz.

<center>

<img alt="" src="../../../rk3576_img/EC-R3576PC/usage_display_interface.png" width="900">
</center>

The RK3576 has 3 Video output ports, and each Video output port is bound to a fixed display controller. The maximum resolutions of each port are as follows:

* Port0 can output up to 4K@120Hz
* Port1 can output up to 2560x1600@60Hz
* Port2 can output up to 1920x1080@60Hz

The SDK defaults to connecting HDMI to Port0 and dp0 to Port2. If dp0 needs to output a higher resolution, dp0 can be reassigned to Port0 (change `dp0_in_vp2` to `dp0_in_vp0` in `rk3576-firefly-roc-rk3576-pc-ext.dtsi`).

### Debugging methods

* Get the edid, supported resolutions and connection status of HDMI/Display Port (taking HDMI as an example):

```
busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/status
```

* Get the information of the Video Portx (and the connected display controller) in use:

```
cat /sys/kernel/debug/dri/0/summary
```

Generally, if the HDMI/Display Port display fails, first run the commands above to check whether the connection status, edid and resolution are correct.

## Ethernet

The EC-R3576PC provides 2 RJ45 ports, corresponding to the `eth0` and `eth1` devices in the system:

<center>

<img alt="" src="../../../rk3576_img/EC-R3576PC/usage_ethernet_interface.jpg" width="900">
</center>

| Hardware device | Driver | Android device name | Primary/Secondary | Speed |
|---|---|---|---|---|
| eth0 | rk_gmac | Ethernet | Primary, for the external network | Gigabit |
| eth1 | r8152 | Ethernet 2 | Secondary, for the internal network | 100Mbps |

**Note**: The secondary port eth1 is a USB2.0 expanded network port, so its maximum speed is 100Mbps.

After both ports are connected to the network, you can check the IP addresses through the debug serial port or adb and perform connectivity tests:

```
ifconfig eth0
ifconfig eth1
ping -I eth0 -c 10 www.baidu.com
ping -I eth1 -c 10 168.168.4.168
```

## Storage (M.2 SATA3.0 / PCIe2.0)

**Note**: The default configuration of the EC-R3576PC does not include a SATA or PCIe storage device.

The EC-R3576PC has 1 M.2 interface, which can be configured in software as an M.2 SATA3.0 interface (supporting SSDs with the SATA protocol) or as an M.2 PCIe2.0 interface (supporting SSDs with the NVMe protocol). The software is configured as an M.2 SATA3.0 interface by default.

<center>

<img alt="" src="../../../rk3576_img/EC-R3576PC/usage_sata_interface.jpg" width="900">
</center>

The switch between SATA and PCIe is controlled by the `M2_SATA_OR_PCIE` macro in the DTS (located in `rk3576-firefly-roc-rk3576-pc.dtsi`): **the default value is 1, which configures SATA3.0. To configure PCIe2.0, change it to 0**.

Mounting and speed test: the device node recognized by the system is generally `/dev/block/sda`. The commands for formatting and mounting are as follows:

```
mkfs.ext4 /dev/block/sda
mount /dev/block/sda /mnt/media_rw/
df -h
```

## RELAY (Relay Output)

The EC-R3576PC supports one relay output, where ON corresponds to OUTPUT1 in the hardware schematic and COM corresponds to RELAY_COM1 in the hardware schematic.

<center>

![](../../../rk3576_img/EC-R3576PC/output_interface.jpg)
</center>

When GPIO3_D0 outputs a low level, OUTPUT1 and RELAY_COM1 are disconnected; when it outputs a high level, OUTPUT1 and RELAY_COM1 are connected. Since the lower monochrome LED Ext Yellow(L2) and the relay are controlled by the same GPIO, controlling this LED is controlling the relay output:

```
echo 1 > /sys/class/leds/extuser/brightness # the lower monochrome led (L2) is on, the relay is connected
echo 0 > /sys/class/leds/extuser/brightness # the lower monochrome led (L2) is off, the relay is disconnected
```

## INPUT (Optocoupler Isolation Input)

The EC-R3576PC supports one optocoupler isolation input, where IN corresponds to INPUT1 in the hardware schematic and G corresponds to INPUT_COM.

<center>

![](../../../rk3576_img/EC-R3576PC/input_interface.jpg)
</center>

When `INPUT(IN)` and `INPUT_COM(G)` are connected, the GPIO detects a low level; when they are disconnected, it detects a high level. The detection commands are as follows:

```
# Request the GPIO
echo 112 > /sys/class/gpio/export
# Set it as input
echo in > /sys/class/gpio/gpio112/direction
# Read the level value
cat /sys/class/gpio/gpio112/value
```

**Note**: If a high-voltage signal (such as 24V) needs to be connected, a resistor with a resistance between 3.9K and 4.7K must be connected in series before INPUT(IN) to prevent the optocoupler isolation chip from being burned out.

## LED

The EC-R3576PC provides 3 LEDs (one tricolor LED and two monochrome LEDs). The LEDs are defined as devices under the `/sys/class/leds/` directory, and you can control on/off by writing to `brightness`, for example:

```
echo 1 > /sys/class/leds/extuser/brightness # light on the lower monochrome led (L2)
echo 0 > /sys/class/leds/extuser/brightness # light off the lower monochrome led (L2)
```

The specific device names depend on the actual output of `ls /sys/class/leds/` in the system.
