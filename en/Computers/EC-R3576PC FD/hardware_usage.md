# Hardware Function Usage
## Debug Serial

The EC-R3576PC-FD does not expose a debug serial port on the enclosure. You need to open the host and connect an external USB to TTL serial module to the debug serial pin row on the mainboard for debugging.

For the connection and usage of the debug serial port, see: [Debug Serial](usb_to_ttl.md).

## Display Interface

The ROC-RK3576-PC provides two display output interfaces, HDMI and Display Port, which support multi-screen duplicate/extended display:

* HDMI: supports the HDMI2.1 protocol with a maximum resolution of 4K@120Hz.
* Display Port: represented as `dp0` in the software, supports the DP TX 1.4a protocol with a maximum resolution of 4K@120Hz.

The RK3576 has 3 Video output ports, and each Video output port is bound to a fixed display controller. The maximum resolutions of each port are as follows:

* Port0 can output up to 4K@120Hz
* Port1 can output up to 2560x1600@60Hz
* Port2 can output up to 1920x1080@60Hz

The SDK defaults to connecting HDMI to Port0 and dp0 to Port2. If dp0 needs to output a higher resolution, dp0 can be reassigned to Port0 (change `dp0_in_vp2` to `dp0_in_vp0` in `rk3576-firefly-roc-rk3576-pc.dtsi`).

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

The ROC-RK3576-PC provides 1 RJ45 Gigabit port, corresponding to the `eth0` device in the system.

After the port is connected to the network, you can check the IP address through the debug serial port or adb and perform a connectivity test:

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```

## Storage (M.2 SATA3.0 / PCIe2.0)

The ROC-RK3576-PC has 1 M.2 interface, which can be configured in software as an M.2 SATA3.0 interface (supporting SSDs with the SATA protocol) or as an M.2 PCIe2.0 interface (supporting SSDs with the NVMe protocol). The software is configured as an M.2 SATA3.0 interface by default.

The switch between SATA and PCIe is controlled by the `M2_SATA_OR_PCIE` macro in the DTS (located in `rk3576-firefly-roc-rk3576-pc.dtsi`): **the default value is 1, which configures SATA3.0. To configure PCIe2.0, change it to 0**.

The device node recognized by the system is generally `/dev/block/sda`. The commands for formatting and mounting are as follows:

```
mkfs.ext4 /dev/block/sda
mount /dev/block/sda /mnt/media_rw/
df -h
```

The computer also has 1 TF card slot, and the TF card can be used in the system after being inserted.
