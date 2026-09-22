# Hardware Function Usage

## Debug Serial Port

The AIBOX-1688 uses an on-board Type-C debug serial port. Simply connect the debug Type-C port of the machine to a USB port of the PC with a Type-C to USB data cable for direct debugging — no DIP switch or other switch operation is required.

The serial port parameters are:

* Baud rate: 115200
* Data bits: 8
* Stop bits: 1
* Parity: none
* Flow control: none

The username and password for terminal login are both `linaro`. For the connection method and usage under Windows (MobaXterm) and Linux (minicom), see: [Serial Port Debugging](debug.md).

## Ethernet

The AIBOX-1688 provides 2 gigabit Ethernet ports on the back:

* Ethernet port 0 (near the power port): obtains an IP address dynamically via DHCP by default
* Ethernet port 1 (near the USB ports): static IP `192.168.150.1` with subnet mask `255.255.255.0` by default

For initial use, set the PC to an IP address in the same subnet (e.g. `192.168.150.2/24`). After confirming that you can `ping` the device, log in remotely with `ssh linaro@192.168.150.1` (username and password are both `linaro`).

### Network IP Configuration (netplan)

In the Ubuntu system, `eth0` of the dual Ethernet ports obtains an IP address dynamically by default (i.e. DHCP), while `eth1` is fixed to `192.168.150.1`.

Network configuration is done through the `/etc/netplan/01-netcfg.yaml` file, which can be used to modify the original network configuration of the Ubuntu system.

```shell
linaro@sophon:~$ cat /etc/netplan/01-netcfg.yaml
network:
        version: 2
        renderer: networkd
        ethernets:
                eth0:
                        dhcp4: yes
                        addresses: []
                        optional: yes
                        dhcp-identifier: mac
                eth1:
                        dhcp4: no
                        addresses: [192.168.150.1/24]
                        optional: yes
```

If the user deletes this file, `eth1` will obtain an IP via DHCP just like `eth0`. If the user wants to set a static IP for `eth0` as well, just modify the `eth0` node in the `/etc/netplan/01-netcfg.yaml` file (referring to `eth1`), and restart for the changes to take effect.

## Display Interface

The AIBOX-1688 provides 1 HDMI 2.0 output on the back, supporting up to 4K@60fps.

You can test the HDMI interface with the following commands:

```shell
insmod /mnt/system/ko/soph_drm.ko # Install display framework driver
systemctl stop SophonHDMI.service # Stop HDMI display interface
systemctl restart SophonHDMI.service  # Restore HDMI display interface
```

You can also test using `modetest`:

```shell
modetest -M cvitek -s 42@40:1920x1080-60@RG24
```

Where `42@40:1920x1080-60@RG24` should be modified according to the actual resolution of the connected monitor.

## USB Interface

The AIBOX-1688 provides 2 USB 3.0 ports on the back (the upper port supports USB 3.0 only, no USB 2.0), and 1 TYPE-C USB 2.0 OTG port on the front.

When a USB drive or other storage device is inserted, it will be recognized as a node like `/dev/sdb1`. The system does not support automatic mounting, so manual `mount` is required:

```shell
# Create mount directory
mkdir disk
# Mount
sudo mount /dev/sdb1 disk
# View files in the USB drive
ls disk
```

After writing data, please use `sync` or `umount` promptly. When shutting down, please use the `sudo poweroff` command instead of a forced power-off to avoid data loss.

## TF Card

The AIBOX-1688 provides a TF card slot on the front (supporting high-speed cards). Mounting a TF card is similar to a USB drive:

```shell
# Create mount directory
mkdir media
# Mount
sudo mount /dev/mmcblk1p1 media
# View files in the TF card
ls media
```

The TF card can also be used for firmware upgrade. See [Firmware Upgrade](fw_upgrade.md).

## SSD

The SSD is located inside the box. If it is not configured at the factory, you need to add it yourself; the factory firmware does not support SATA SSD. To support it, the Linux kernel needs to be replaced.

Generally, the mountable PCIE node is: `/dev/nvme0n1p1`.

```shell
# Create mount directory
mkdir nvme_dir
# Mount
sudo mount /dev/nvme0n1p1 nvme_dir
# View files in the PCIE SSD
ls nvme_dir
```

## Fan

The fan operation of the AIBOX-1688 is divided into 4 levels:

| Current Temperature of Main Control | Fan Operation Level |
|:----------------------------------:|:-------------------:|
| ≥ 33℃ | 1 |
| ≥ 45℃ | 2 |
| ≥ 55℃ | 3 |
| ≥ 60℃ | 4 |

The higher the fan operation level, the faster the speed. If the current temperature of the main control is below 33℃, the fan is off by default.

Users can check the current operating temperature of the main control by executing the following command:

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

For example, the following shows 45.5 ℃:

```shell
linaro@bm1688:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

In addition, users can also manually turn on the fan (write 0-4, where 0 is off and 4 is the maximum level):

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```

## Memory Allocation Adjustment

A portion of the memory of the AIBOX-1688 is allocated to the NPU, VPP and VPU devices, so the memory data obtained by commands such as `free -h` is inconsistent with the actual memory of the machine.

If you need to view the detailed memory allocation settings, or the default memory allocation conflicts with actual needs and needs to be adjusted, please refer to the following steps.

### Install the Memory Allocation Tool

Obtain the memory allocation tool package from the [Download Center](https://community.t-firefly.com/doc/download/266) and transfer it to the AIBOX-1688, then execute the following command to unpack it:

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

### View Memory Allocation Settings

After unpacking the package, execute the following command to view the memory allocation settings:

```bash
cd memory_edit
./memory_edit.sh -p
```

An example of the memory allocation settings result is as follows, where `Info: get max memory size ...` shows the maximum configurable memory of each device, and `Info: get now memory size ...` shows the memory currently allocated to each device.

```bash
linaro@aibox-1688x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1688x_se7_v1_mini.dts
Info: =======================================================================
Info: get ddr information ...
Info: ddr12_size 8589934592 Byte [8192 MiB]
Info: ddr3_size 4294967296 Byte [4096 MiB]
Info: ddr4_size 4294967296 Byte [4096 MiB]
Info: ddr_size 16384 MiB
Info: =======================================================================
Info: get max memory size ...
Info: max npu size: 0x1dbf00000 [7615 MiB]
Info: max vpu size: 0xc0000000 [3072 MiB]
Info: max vpp size: 0x100000000 [4096 MiB]
Info: =======================================================================
Info: get now memory size ...
Info: now npu size: 0xf6e00000 [3950 MiB]
Info: now vpu size: 0x80000000 [2048 MiB]
Info: now vpp size: 0xc0000000 [3072 MiB]
```

### Modify Memory Allocation Settings

You can refer to the following commands to configure the memory allocation, where the three input parameters are decimal numbers (unit MiB) of the sizes to be configured for NPU, VPU and VPP; or hexadecimal values (unit Byte).

```bash
# Decimal, unit MiB
./memory_edit.sh -c -npu 2048 -vpu 2048 -vpp 2048
# Hexadecimal, unit Byte
./memory_edit.sh -c -npu 0x80000000 -vpu 0x80000000 -vpp 0x80000000
```

After executing one of the above commands, check whether there is any Error in the output, and whether the sizes of the three parts in the output similar to the following are the same as the required configuration.

```bash
Info: output configuration results ...
Info: vpu mem area(ddr3): 0x8000000 [128 MiB] 0x20000000 -> 0x27ffffff
Info: ion npu mem area(ddr1): 0x80000000 [2048 MiB] 0x24100000 -> 0xa40fffff
Info: ion vpu mem area(ddr3): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: ion vpp mem area(ddr4): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: =======================================================================
Info: start check memory size ...
Info: check npu size: 0x80000000 [2048 MiB]
Info: check vpu size: 0x80000000 [2048 MiB]
Info: check vpp size: 0x80000000 [2048 MiB]
Info: check edit size ok
Info: en_emmcfile ok
```

If the check is correct, please save your current work, and follow the steps below to replace the boot image in the boot partition with the modified emmcboot.itb file, and finally reboot the machine to make the changes take effect.

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

## Power On and Power Off

The AIBOX-1688 will turn on automatically when connected to the power supply. If the power supply is connected and the machine was shut down by command, please short press the power button to turn it on.

When the board boots into the Linux system, the LED will be on with green color.

Note: Please complete the software/hardware shutdown before disconnecting the power, so as not to damage the filesystem data.

* Software shutdown: run `sudo poweroff` in the terminal.
* Button shutdown: press and hold the power button until the work LED stops flashing.

When the fan stops running and the indicator LED is off, the AIBOX-1688 has completed shutdown, and the power supply can be safely disconnected.

## Debugging and FAQs

**What are the default username and password of the system?**

* Username: `linaro`
* Password: `linaro`
* Switch to superuser: `sudo -s`

**What should I do if the system fails to boot and keeps rebooting?**

It may be caused by insufficient power current. Please use a power supply with a voltage of 12V and a current of 5A or more.

**Checking memory:**

* System memory: `free -h`
* ION memory
    * NPU : `cat /sys/kernel/debug/ion/cvi_npu_heap_dump/summary`
    * VPP : `cat /sys/kernel/debug/ion/cvi_vpp_heap_dump/summary`
