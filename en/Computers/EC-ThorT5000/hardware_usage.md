# Hardware Function Usage
## Debug Serial

The EC-ThorT5000 has an onboard Type-C debug serial port. Just connect the device to the PC directly with a Type-C cable for serial debugging. The baud rate is 115200, and no extra USB-to-serial adapter is required.

## Serial debug

USB-to-serial adapter is the abbreviation of USB-to-serial TTL adapter.

### Debugging 

You can connect EC-ThorT5000 to a PC for serial port debugging:

<center>

![](../../../nvidia_img/EC-ThorT5000/type-c_connection.png)
</center>

#### Serial parameter configuration

EC-ThorT5000 uses the following serial port parameters:

* Baud rate: 115200
* Data bits: 8
* Stop bit: 1
* Parity: none
* Flow Control: None

#### Using serial port debugging on Windows

On Windows, putty or SecureCRT is generally used. Among them, we recommend using the free version of MobaXterm. This is a powerful terminal software, which is introduced here. The usage of other software is similar.

Go here [download MobaXterm](https://mobaxterm.mobatek.net/):

1. Select `session` as `Serial`.
2. Modify `Serial port` to the COM port found in Device Manager.
3. Set `Speed (bsp)` to `115200`.
4. Click the `OK` button.

<center>

<img alt="" src="../../../nvidia_img/debug_set_MobaXterm1.PNG" width="800">
</center>
<center>

<img alt="" src="../../../nvidia_img/debug_set_MobaXterm2.PNG" width="800">
</center>

#### Serial debugging on Linux

There are several options available on Linux:

* minicom
* picocom
* kermit

The following will introduce the use of minicom.

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

The above prompt `CTRL-A Z` is the escape key, press `Ctrl-a` and then `z` to bring up the menu:

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

Press `O` according to the prompt to enter the setting interface, as follows:

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

Move the cursor to "Serial port setup", press enter to enter the serial port setup interface, then enter the letters indicated above, select the corresponding option, and set as follows:

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


## Network

The EC-ThorT5000 supports Ethernet. The default network card names in the system are as follows:

* Gigabit Ethernet: `enP2p1s0`
* 10Gbps Ethernet: `mgbe0`, `mgbe1`, `mgbe2`, `mgbe3`

For the network configuration, please refer to [Network Configuration](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#network-configuration).

## SIM Card

The SIM card slot of the EC-ThorT5000 is used together with a 4G/5G module to provide mobile network connectivity. **The 4G/5G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../nvidia_img/EC-ThorT5000/sim_insert_direction.png" width="400">
</center>

## CAN

CAN (Controller Area Network) is a kind of serial communication network which can effectively support distributed control or real-time control. See the [TI application report](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf) for more details.

The 4 CAN interfaces of the **CAN Version** are led out from the 24Pin Phoenix terminal block. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L.

On Ubuntu, run `apt update && apt install can-utils` first to install the test tools. The communication test commands are as follows (taking `can0` as an example):

```
# Close the can0 device
ip link set can0 down
# Set the bitrate to 250Kbps
ip link set can0 type can bitrate 250000
# Open the can0 device
ip link set can0 up
# Run candump on the receiving end, blocking and waiting for frames
candump can0
# Run cansend on the sending end to send a frame
cansend can0 123#1122334455667788
```

If no frames are received after sending, please check whether the bus CAN_H and CAN_L are loose or reversed.

## IO

### INPUT

```
sudo su
gpioget `gpiofind "PAL.01"`
```

### OUTPUT

Set high:

```
sudo su
gpioset `gpiofind "PT.06"`=1
```

Set low:

```
sudo su
gpioset `gpiofind "PT.06"`=0
```

## UART (RS232 / RS485)

The **CAN Version** leads out the RS232 and RS485 interfaces through the 24Pin Phoenix terminal block. For the pin definitions, see the Interface Description in [Product Introduction](started.md). When wiring, the TXD/RXD of the RS232 need to be cross-connected to the peer device, and the 485_A/485_B of the RS485 are connected to the A/B of the peer device correspondingly.

You can use `ls /dev/tty*` to check the serial device nodes in the system. For sending and receiving tests, you can use the `cat` and `echo` commands or serial terminal tools such as minicom (the tool usage is the same as that in the Debug Serial section).

The **Ethernet Version** does not lead out RS232/RS485 interfaces.

## Watchdog

The EC-ThorT5000 has 1 external watchdog, corresponding to `/dev/wdt_crl`. Usage:

```bash
# Enable the watchdog
echo e > /dev/wdt_crl

# Set the timeout, there are 4 available values
echo 0 > /dev/wdt_crl # 0.64 sec
echo 1 > /dev/wdt_crl # 2.56 sec
echo 2 > /dev/wdt_crl # 10.24 sec
echo 3 > /dev/wdt_crl # 40.96 sec

# Disable the watchdog
echo d > /dev/wdt_crl
```

## Audio

Users can output audio through the headphone jack and the HDMI port. You can switch between the headphone and HDMI interfaces in the system settings, selecting one for output.

In the terminal, execute `cat /proc/asound/cards` to view the sound cards: `HDA` represents the HDMI sound card, and `APE` represents the headphone sound card.

HDMI output:

```shell
aplay -D hw:HDA,3 hdmi.wav # hdmi.wav needs to be a dual-channel audio file
```

Headphone output:

```shell
aplay -D hw:APE,0 test.wav
```

Audio input (recording via the headphone jack):

```shell
arecord -D hw:APE,0 -c 2 -r 16000 -f S32_LE test.wav
```

## Storage

The EC-ThorT5000 supports various storage device interfaces (USB, TF card, etc.). After a storage device is inserted, it will be recognized as a node similar to `/dev/sdb1`, `/dev/nvme0n1p1` or `/dev/mmcblk1p1`, which is the same as in the desktop PC Linux environment. The file systems supported include FAT, FAT32, EXT2/3/4 and NTFS. The device does not support automatic mounting, so you need to mount it manually with `mount`. For example, to mount a USB drive:

```shell
# create mount directory
mkdir disk
# mount
sudo mount /dev/sdb1 disk
# view the files in the USB drive
ls disk
```

The mounting of TF cards and PCIe SSDs is similar to that of the USB drive; the corresponding nodes are subject to the actual output of `dmesg`. After writing data, please use `sync` or `umount` in time, and use the `sudo poweroff` command when shutting down to avoid data loss.

## Display

The EC-ThorT5000 provides 4 HDMI2.0 display output interfaces (up to 4K@60Hz); the display works after the monitor is connected. It also provides 8 channels of GMSL2 interfaces, which can be connected to GMSL2 camera modules for video input.
