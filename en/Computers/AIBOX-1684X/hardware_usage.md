# Hardware Function Usage
## Debug Serial Port

The AIBOX-1684X uses an on-board Type C debug serial port. Just connect the Type C port of the machine to the USB port of the PC with a Type C-to-USB data cable for direct debugging. Both the login account and password are `linaro`.

The serial port parameters are:

* Baud rate: 115200
* Data bits: 8
* Stop bit: 1
* Parity: none
* Flow Control: None

### Using serial port debugging on Windows

On Windows, putty or SecureCRT is generally used. We recommend using the free version of MobaXterm. It is a powerful terminal software, and the usage of other serial software is similar.

Go here to [download MobaXterm](https://mobaxterm.mobatek.net/):

1. Select `session` as `Serial`.
2. Modify `Serial port` to the COM port found in Device Manager.
3. Set `Speed (bsp)` to `115200`.
4. Click the `OK` button.

<center>

<img alt="" src="../../../bm1684_img/debug_set_MobaXterm1.PNG" width="800">
</center>
<center>

<img alt="" src="../../../bm1684_img/debug_set_MobaXterm2.PNG" width="800">
</center>

### Serial debugging on Linux

There are several options available on Linux: minicom, picocom, kermit. Due to space constraints, the use of minicom is introduced below.

Install minicom:

```
sudo apt-get install minicom
```

After connecting the serial cable, see what the serial device file is. The following example is `/dev/ttyUSB0`:

```
$ ls /dev/ttyUSB*
/dev/ttyUSB0
```

Run:

```
$ sudo minicom
Welcome to minicom 2.7
OPTIONS: I18n
Compiled on Jan  1 2014, 17:13:19.
Port /dev/ttyUSB0, 15:57:00
Press CTRL-A Z for help on special keys
```

The above prompt `CTRL-A Z` is the escape key. Press `Ctrl-a` and then `z` to bring up the menu:

```
   +-------------------------------------------------------------------+
                         Minicom Command Summary                      |
  |                                                                    |
  |              Commands can be called by CTRL-A <key>                |
  |                                                                    |
  |               Main Functions                  Other Functions      |
  |                                                                    |
  | Dialing directory..D  run script (Go)....G | Clear Screen.......C  |
  | Send files.........S  Receive files......R | cOnfigure Minicom..O  |
  | comm Parameters....P  Add linefeed.......A | Suspend minicom....J  |
  | Capture on/off.....L  Hangup.............H | eXit and reset.....X  |
  | send break.........F  initialize Modem...M | Quit with no reset.Q  |
  | Terminal settings..T  run Kermit.........K | Cursor key mode....I  |
  | lineWrap on/off....W  local Echo on/off..E | Help screen........Z  |
  | Paste file.........Y  Timestamp toggle...N | scroll Back........B  |
  | Add Carriage Ret...U                                               |
  |                                                                    |
  |             Select function or press Enter for none.               |
  +--------------------------------------------------------------------+
```

Press `O` according to the prompt to enter the setting interface:

```
           +-----[configuration]------+
           | Filenames and paths      |
           | File transfer protocols  |
           | Serial port setup        |
           | Modem and dialing        |
           | Screen and keyboard      |
           | Save setup as dfl        |
           | Save setup as..          |
           | Exit                     |
           +--------------------------+
```

Move the cursor to `Serial port setup`, press enter to enter the serial port setup interface, then enter the letters indicated above, select the corresponding option, and set as follows:

```
   +-----------------------------------------------------------------------+
   | A -    Serial Device      : /dev/ttyUSB0                              |
   | B - Lockfile Location     : /var/lock                                 |
   | C -   Callin Program      :                                           |
   | D -  Callout Program      :                                           |
   | E -    Bps/Par/Bits       : 115200 8N1                                |
   | F - Hardware Flow Control : No                                        |
   | G - Software Flow Control : No                                        |
   |                                                                       |
   |    Change which setting?                                              |
   +-----------------------------------------------------------------------+
```

**Note:** `Hardware Flow Control` and `Software Flow Control` must be set to No, otherwise it may result in failure to input.

After the setup is complete, go back to the previous menu and select `Save setup as dfl` to save it as the default configuration, which will be used by default in the future.

## Login and Network Configuration

### Login

The AIBOX-1684X supports two login methods: debug serial port login and Ethernet SSH login. Both the login account and password are `linaro`.

* Debug serial port login: see "Debug Serial Port" above for usage.
* Ethernet SSH login: The AIBOX-1684X provides 2 Gigabit Ethernet interfaces. By default, network port 0 (near the USB ports) is set with a dynamic IP, while network port 1 (near the 12V power interface) is set with the static IP `192.168.150.1` and subnet mask `255.255.255.0`. For the initial login, it is recommended to select network port 1. First set the PC to an IP address in the same network segment, such as `192.168.150.2/24`. After the network port light flashes normally, open the terminal and use `ssh` to log in, with port number `22`:

  ```shell
  ssh linaro@192.168.150.1
  ```

If the login fails, you can try to use the PC to `ping` the IP address of network port 1 of the AIBOX-1684X. Note that the PC needs to add an IP address in the same network segment, because the IP address of the PC is not necessarily in the same network segment.

The following is how to add a same-network-segment IP on the PC (using administrator mode):

- Windows 10

  ```
  netsh int ipv4 add address "Ethernet" 192.168.150.101 255.255.255.0
  ```
  Where `"Ethernet"` refers to the network interface, `192.168.150.101` is the IP address, and `255.255.255.0` is the subnet mask.

- Linux

  ```shell
  ifconfig enp4s0:1 192.168.150.89
  ```
  Where `enp4s0` is the name of the network card driver of the PC, which can be viewed by executing the `ifconfig` command.

### Network IP configuration

By default, the AIBOX-1684X has a dynamic IP set for network port 0, and a static IP set for network port 1. These configurations are implemented through the `/etc/netplan/01-netcfg.yaml` file in the system.

```shell
linaro@aibox-1684x:~$ cat /etc/netplan/01-netcfg.yaml
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

Users can modify the network configuration by modifying the `/etc/netplan/01-netcfg.yaml` file, for example:

- Set network port 0 to a static IP: Refer to the original setting of `eth1` to modify the `eth0` node, change the `dhcp4` parameter to `no`, and add the IP address to the `addresses` parameter.

- Set network port 1 to a dynamic IP: Refer to the original setting of `eth0` to modify the `eth1` node, change the `dhcp4` parameter to `yes`, and remove the IP address in the `addresses` parameter.

After modifying the `/etc/netplan/01-netcfg.yaml` file, users can make the settings take effect immediately by executing the `sudo netplan apply` command.

## USB Interfaces

The AIBOX-1684X provides 2 USB 3.0 interfaces. After inserting a USB flash drive, the storage device will be recognized as a node similar to `/dev/sdb1`. The system does not support auto-mounting, so you need to mount it manually with the `mount` command.

Insert the USB drive, and use `dmesg` to know the `sd` device corresponding to the USB drive:

```shell
[15460.953423] [5]  sdb: sdb1
```

Mount the USB drive:

```shell
# create mount directory
mkdir disk
# mount
sudo mount /dev/sdb1 disk
# View the files in the USB drive
ls disk
```

After completing data writing, please use `sync` or `umount` in time, and please use the `sudo poweroff` command when shutting down to avoid violent power-off, so as to avoid data loss.

## TF Card

The AIBOX-1684X provides 1 TF card slot, which can be used to mount a TF card for storage expansion, as well as to upgrade the system firmware via a TF card.

The TF card mounting is similar to the USB drive. Use `dmesg` to know the `mmcblk` device corresponding to the TF card:

```shell
[16220.776440] [4]  mmcblk1: p1
```

Mount the TF card:

```shell
# create mount directory
mkdir media
# mount
sudo mount /dev/mmcblk1p1 media
# View the files in the TF card
ls media
```

For firmware upgrade via TF card, see [System Firmware Upgrade](fw-upgrade-by-sdcard.md).

## Fan

The AIBOX-1684X fans work in 5 levels:

| Main control current temperature | Fan working level |
|:----------:|:----------:|
| ≥ 33℃ | 1 |
| ≥ 55℃ | 2 |
| ≥ 60℃ | 3 |
| ≥ 65℃ | 4 |
| ≥ 70℃ | 5 |

The higher the working level of the fan, the faster the speed. If the current temperature of the main controller is lower than 33℃, the fan is turned off by default.

Users can view the current working temperature of the main controller by executing the following command:

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

For example, the following is 45.5℃:

```shell
linaro@bm1684:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

In addition, the user can also manually turn on the fan (write 0-5, 0 is off, 5 is the maximum level):

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```

## Memory

A portion of the memory of the AIBOX-1684X is allocated to NPU, VPP, and VPU devices. Therefore, the memory data obtained using commands such as `free -h` is inconsistent with the actual memory of the AIBOX-1684X.

If you need to view the detailed memory allocation settings, or if the default memory allocation conflicts with actual requirements and needs to be adjusted, you can refer to the following steps.

### Install the memory allocation tool

Obtain the memory allocation tool package from [Download Center] and transfer it to the AIBOX-1684X. Run the following command to decompress the package:

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

### View memory allocation settings

After decompressing the package, run the following command to view the memory allocation settings:

```bash
cd memory_edit
./memory_edit.sh -p
```

The following is an example of the memory allocation settings. The result displayed under `Info: get max memory size ...` is the maximum memory that can be configured for each device, and the result displayed under `Info: get now memory size ...` is the memory currently allocated for each device.

```bash
linaro@aibox-1684x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1684x_se7_v1_mini.dts
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

### Modify memory allocation settings

You can set the memory allocation by referring to the following commands. The three input parameters are decimal numbers of the sizes to be configured for NPU, VPU, and VPP in MiB; or hexadecimal values in Byte.

```bash
# decimal, in MiB
./memory_edit.sh -c -npu 2048 -vpu 2048 -vpp 2048
# hexadecimal, in bytes
./memory_edit.sh -c -npu 0x80000000 -vpu 0x80000000 -vpp 0x80000000
```

After executing one of the above commands, check whether there is an Error in the output, and whether the sizes of the three parts in the output similar to the following are the same as the sizes to be configured.

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

If the check is correct, save the current work, replace the boot image in the boot partition with the modified `emmcboot.itb` file by referring to the following operations, and finally restart the machine for the changes to take effect.

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

## FAQs

### What is the default username and password of the system?

* Username: `linaro`
* Password: `linaro`
* Switch to superuser: `sudo -s`

### What should I do if the startup is abnormal and the system restarts repeatedly?

It may be that the power supply current is not enough. Please use a power supply with a voltage of 12V and a current of more than 5A.

[Download Center]: https://community.t-firefly.com/en/doc/download/248