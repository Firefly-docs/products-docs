# Hardware Function Usage
## Debug Serial Port

The EC-A186JD4 has an on-board Type-C debug serial port. Simply connect the Type-C port of the machine to a USB port of the PC with a Type-C to USB data cable for direct debugging.

The serial port parameters are:

* Baud rate: 115200
* Data bits: 8
* Stop bits: 1
* Parity: none
* Flow control: none

The username and password for terminal login are both `linaro`.

## Display Interface

The EC-A186JD4 provides 1 HDMI output interface, supporting standard HDMI 2.0 output.

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

## UART

The on-board RS232 serial port of EC-A186JD4 corresponds to the device node `/dev/ttyS2` with a default baud rate of `115200`.

### Serial Loopback Test

After shorting the TXD and RXD pins of the serial port, run the following commands on the device:

```shell
stty -F /dev/ttyS2 115200
cat /dev/ttyS2 &
echo "firefly test" > /dev/ttyS2
```

If the terminal outputs `firefly test`, the serial transceiving works normally.



## Login and Network Configuration

In addition to logging in through the Type-C debug serial port (see the Debug Serial Port section above), the EC-A186JD4 provides 2 gigabit Ethernet ports and also supports remote login via Ethernet SSH.

### Ethernet SSH Login

By default, at the factory, port 0 (near the USB connector) of the EC-A186JD4 is set to a dynamic IP, while port 1 (near the HDMI port) is set to a static IP `192.168.150.1` with a subnet mask of `255.255.255.0`. You can set the PC to `192.168.150.2/24` for the initial login.

The IP address of port 0 is assigned by a router and is generally unknown in advance, so it is best to choose port 1 for the initial login.

After the port LEDs blink normally, open a terminal and log in with `ssh`. The port number is `22`, and the username and password are both `linaro`:

```shell
ssh linaro@192.168.150.1
```

If the login fails, you can check whether the PC can `ping` the IP address of port 1 of the EC-A186JD4. Note that the PC must have an IP address in the same subnet, because the PC's IP address may not be in the same subnet.

Here is how to add a same-subnet IP address on the PC (in administrator mode):

- Windows 10

  ```
  netsh int ipv4 add address "Ethernet" 192.168.150.101 255.255.255.0
  ```
  Here, `"Ethernet"` refers to the network interface, `192.168.150.101` is the IP address, and `255.255.255.0` is the subnet mask.

- Linux

  ```shell
  ifconfig enp4s0:1 192.168.150.89
  ```
  Here, `enp4s0` is the name of the PC's network card driver, which can be checked by executing the `ifconfig` command.

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

## USB Interface

The EC-A186JD4 provides 2 USB 3.0 ports.

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

The EC-A186JD4 provides a TF card slot. Mounting a TF card is similar to a USB drive:

```shell
# Create mount directory
mkdir media
# Mount
sudo mount /dev/mmcblk1p1 media
# View files in the TF card
ls media
```

The TF card can also be used for firmware upgrade. See [Firmware Upgrade](fw_upgrade.md).

## SIM Card

The SIM card slot of the EC-A186JD4 is used together with a 4G module to provide mobile network connectivity. **The 4G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../bm1688_img/EC-A186JD4/sim_insert_direction.png" width="400">
</center>

## Antenna Connection

The antenna connection of EC-A186JD4 is shown below:

<center>

<img alt="" src="../../../bm1688_img/EC-A186JD4/antenna_connection.jpg" width="400">
</center>



## NVMe SSD

The EC-A186JD4 provides an NVME interface (PCIE3.0 x 1). Generally, the mountable PCIE node is: `/dev/nvme0n1p1`.

```shell
# Create mount directory
mkdir nvme_dir
# Mount
sudo mount /dev/nvme0n1p1 nvme_dir
# View files in the PCIE SSD
ls nvme_dir
```

## Memory Allocation Adjustment

A portion of the memory of the EC-A186JD4 is allocated to the NPU, VPP and VPU devices, so the memory data obtained by commands such as `free -h` is inconsistent with the actual memory of the machine.

If you need to view the detailed memory allocation settings, or the default memory allocation conflicts with actual needs and needs to be adjusted, please refer to the following steps.

### Install the Memory Allocation Tool

Obtain the memory allocation tool package from the [Download Center](https://community.t-firefly.com/en/doc/download/250) and transfer it to the EC-A186JD4, then execute the following command to unpack it:

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

## Serial Port (RS232 / RS485)

The EC-A186JD4 has an RS485 interface, with the device name `/dev/ttyS4`, supporting half-duplex, and a default baud rate of `9600`.

There is also an RS232 interface, with the device name `/dev/ttyS2`, supporting full-duplex, and a default baud rate of `115200`.

For the interface location, please refer to the machine figure at the top of this page.

### RS485 Debugging Method

Users can use different USB-to-serial adapters on the host to send and receive data to the device's serial port. For example, the debugging steps for RS485 are as follows:

(1) Connect the hardware

Connect the A, B and GND pins of the RS485 port of the EC-A186JD4 to the A, B and GND pins of the host serial port adapter (USB to 485 serial port module) respectively.

(2) Open the serial terminal on the host

Open kermit in the terminal and set the baud rate:

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file of the USB-to-serial adapter

(3) Send data

The device file of RS485 is `/dev/ttyS4`. Run the following commands on the device (**Note: before running the commands, you need to enable the RS485 sending function, because the device has a GPIO switch that toggles RS485 between the sending state and the receiving state.**):

```
sudo -s
echo 1 > /sys/class/leds/RS485_H_SEND_L_RECV/brightness # 1 -> TX, 0 -> RX
stty -F /dev/ttyS4 9600 -echo
echo firefly RS485 test... > /dev/ttyS4
```

The serial terminal on the host will receive the string "firefly RS485 test..."

(4) Receive data

First run the following commands on the device:

```
sudo -s
cat /dev/ttyS4
```

Then enter the string "Firefly RS485 test..." in the serial terminal of the host, and the same string can be seen on the device side.

### RS232 Debugging Method

The testing method is similar to the RS485 steps; just pay attention to the device name and baud rate. The only difference is that RS232 **does not** require switching between the sending and receiving states.

## CAN

### Introduction to CAN

CAN (Controller Area Network) bus is a serial communication network that effectively supports distributed control or real-time control. The CAN bus is a bus protocol widely used in automobiles, designed for communication between microcontrollers in automotive environments. For more information, please refer to the [CAN Application Report](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf).

### Hardware Connection

The CAN interface location of the EC-A186JD4 can be found in the machine figure at the top of this page. Since there is only one CAN, the first device created in the kernel is `can0` by default.

### CAN Communication Test

Use the candump and cansend tools to test sending and receiving messages:

```
# Close the can0 device on both the receiving and sending ends
ip link set can0 down
# Set the bitrate to 250Kbps on both the receiving and sending ends
ip link set can0 type can bitrate 250000
# Open the can0 device on both the receiving and sending ends
ip link set can0 up
# Run candump on the receiving end, blocking and waiting for messages
candump can0
# Run cansend on the sending end to send a message
cansend can0 123#1122334455667788
```

### FAQS

A summary of several problems encountered during debugging and their solutions:

#### The message is received a long time after sending, or is not received at all.

Check the CAN_H and CAN_L bus, and whether the DuPont wires are loose or reversed.

## Audio (Headphone)

The EC-A186JD4 supports the American standard headphone interface.

Playback Test:

```
$ aplay -D hw:CARD=cv186xdac,DEV=1 /usr/share/sounds/alsa/Front_Center.wav
```

Recording Test:

```
$ arecord -D hw:CARD=cv186xadc,DEV=0  -c 2 -r 48000  -f S16_LE test.wav # Record audio (mono)
$ aplay -D hw:CARD=cv186xdac,DEV=1  test.wav # Play the recorded audio
```

## Debugging and FAQs

**What are the default username and password of the system?**

* Username: `linaro`
* Password: `linaro`
* Switch to superuser: `sudo -s`

**What should I do if the system fails to boot and keeps rebooting?**

It may be caused by insufficient power current. Please use a power supply with a voltage of 12V and a current of 5A or more.

