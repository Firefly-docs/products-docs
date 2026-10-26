# Hardware Function Usage

## Login

There are two ways to log in to the AIBOX PRO KIT: one is terminal login through the Console (Debug Serial), and the other is login through HDMI.

### Console Login
Connect the Type-C cable to the Console port. The login account is `root`, and no `root` password is set by default.<br>
Please refer to the [Hardware Interface Introduction](interface_definition.md).
Use the following serial port parameters:
* Baud rate: 115200
* Data bits: 8
* Stop bits: 1
* Parity check: None
* Flow control: None

#### Serial Debugging on Windows

After connecting the board to the computer with a Type-C cable, the system will prompt that new hardware is found and complete the initialization. Then you can find the corresponding COM port in Device Manager:

<center>

<img alt="" src="../../../modules_img/TypeC-Serial-Debug/debug_find_com.png" width="800">
</center>

On Windows, putty or SecureCRT is generally used. Here we recommend the free version of MobaXterm, which is a powerful terminal software. Other serial port software is used in a similar way.

1. Select `Serial` as the `session` type.
2. Set the `Serial port` to the COM port found in Device Manager.
3. Set `Speed (bsp)` to `115200`.
4. Click the `OK` button.

<center>

<img alt="" src="../../../modules_img/TypeC-Serial-Debug/debug_set_MobaXterm1.png" width="800">
</center>

<center>

<img alt="" src="../../../modules_img/TypeC-Serial-Debug/debug_set_MobaXterm2.png" width="800">
</center>

#### Serial Debugging on Ubuntu

Install minicom:

```
sudo apt-get install minicom
```

Use `minicom -s` to open the configuration interface, enter `Serial port setup`, and set the serial port parameters to `115200 8N1`:

* `E - Bps/Par/Bits`: `115200 8N1`
* `F - Hardware Flow Control`: `No`
* `G - Software Flow Control`: `No`

**Note:** Both `Hardware Flow Control` and `Software Flow Control` must be set to No, otherwise input may become impossible.

After the configuration is completed, select `Save setup as dfl` to save it as the default configuration. After exiting, minicom will connect to the debug serial port at 115200-8-N-1. Log in with the `root` account (no password is set by default).

### HDMI Login
When logging in via the HDMI, it automatically logs in as the user `firefly`, with the password also being `firefly`.

## Watchdog

#### Main Module
The device names for the external watchdogs is `/dev/wdt_core`. For example, using `/dev/wdt_core`, the procedure is as follows:
```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively.
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_core //Enable
echo 2 >/dev/wdt_core //Set the timeout to 10.24 seconds.
```

#### Main Board
The device names for the external watchdogs is `/dev/wdt_base`. For example, using `/dev/wdt_base`, the procedure is as follows:
```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively.
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_base //Enable
echo 2 >/dev/wdt_base //Set the timeout to 10.24 seconds.
```

## RTC

### Introduction

AIBOX PRO KIT has an external RTC powered by a capacitor on the external motherboard, which ensures the RTC continues to run for a short period after power loss. In the kernel, it is represented as `rtc0`.
### Interface Usage

Linux provides three user-space interfaces for interacting with the RTC. The paths are:

*    SYSFS interface: /sys/class/rtc/rtc0/
*    PROCFS interface: /proc/driver/rtc
*    IOCTL interface: /dev/rtc0

Synchronize the current system time to the RTC.
```
hwclock -w
```

#### SYSFS interface
You can directly use `cat` and `echo` to operate the interfaces under /sys/class/rtc/rtc0/.

For example, to view the current RTC date and time:
```
# cat /sys/class/rtc/rtc0/date
2024-07-10
# cat /sys/class/rtc/rtc0/time
02:36:30

```

#### PROCFS interface

To print RTC-related information:
```
# cat /proc/driver/rtc
rtc_time        : 02:37:00
rtc_date        : 2024-07-10
alrm_time       : 00:00:00
alrm_date       : 1970-01-01
alarm_IRQ       : no
alrm_pending    : no
update IRQ enabled      : no
periodic IRQ enabled    : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

#### IOCTL interface

You can use `ioctl` to control `/dev/rtc0`。

For detailed usage instructions, please refer to document `kernel-jammy-src/Documentation/admin-guide/rtc.rst` 。

## RS485

AIBOX PRO KIT has one RS485 interface. If the CPU is RK3588, the device name is `/dev/ttyS6`; if the CPU is RK3576, the device name is `/dev/ttyS3`. It supports half-duplex communication, and the default baud rate is `9600`. This interface uses a Phoenix terminal block, so a compatible terminal connector is required. Once connected, it can be debugged using standard serial port methods.

## CAN

### CAN Introduction

CAN (Controller Area Network) is a serial communication network that effectively supports distributed or real-time control. CAN bus is widely used in automobiles, and it was designed for microcontroller communication in automotive environments. For more information, refer to the [CAN Application Report](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf).

### Hardware Connection

The AIBOX PRO KIT provides one CAN interface on a Phoenix terminal block. When wiring, connect CAN_H to CAN_H and CAN_L to CAN_L. For the interface location, see [Hardware Interface](interface_definition.md).

Since there is only one CAN interface, the first device created in the kernel is `can0` by default.

### CAN Communication Test

Use `candump` and `cansend` to send and receive messages. These tools are included in the SDK and can also be downloaded from [GitHub](https://github.com/linux-can/can-utils).

```
# Bring down can0 on both ends
ip link set can0 down
# Set the bitrate to 250Kbps on both ends
ip link set can0 type can bitrate 250000
# Bring up can0 on both ends
ip link set can0 up
# Run candump on the receiving end, blocking and waiting for frames
candump can0
# Run cansend on the sending end to send a frame
cansend can0 123#1122334455667788
```

The bitrates of both communication ends must be configured identically, otherwise no frames can be received.

#### More Commands

```
1. ip link set canX down                      //Bring down the CAN device
2. ip link set canX up                        //Bring up the CAN device
3. ip -details link show canX                 //Show detailed CAN device information
4. ip link set canX type can bitrate 250000   //Set the CAN bitrate
5. candump canX                               //Receive data from the CAN bus
6. cansend canX --identifier=ID#data          //Send data
7. candump canX,ID:mask                       //Receive frames with matching IDs using a filter
```

### FAQ

#### Messages are received long after sending, or not received at all

Check whether the CAN_H and CAN_L bus wires are loose or connected in reverse.

## DI/DO Optocoupler Isolation

The AIBOX PRO KIT provides one optocoupler-isolated digital input (DI) and one optocoupler-isolated digital output (DO) on the Phoenix terminal block. The mapping between the terminal pins and the SoC pins is as follows:

| Terminal pin | SoC pin | Description |
| --- | --- | --- |
| DI | GPIO47 | Digital input |
| DI_COM | - | Reference ground (level reference) of DI |
| DO | GPIO46 | Digital output |
| DO_COM | - | Reference ground (level reference) of DO |

### DI (Digital Input)

DI_COM is the reference level of DI. When a high level is applied to DI relative to DI_COM, the optocoupler turns on and GPIO47 on the SoC side is pulled low, so the value read is `0`. When there is no input between DI and DI_COM, GPIO47 stays high and the value read is `1`.

To read the DI level (taking GPIO47 as an example):

```
# Export GPIO47 and configure it as an input
echo 47 > /sys/class/gpio/export
echo in > /sys/class/gpio/gpio47/direction
# Read the level: 0 is read when a high level is applied to DI
cat /sys/class/gpio/gpio47/value
```

### DO (Digital Output)

DO_COM is connected to the external ground. DO must be pulled up to an external supply (VCC) through a pull-up resistor, and the data is read from DO. When GPIO46 on the SoC side outputs a low level, the optocoupler turns on and DO is pulled low to the DO_COM level; when GPIO46 outputs a high level, the optocoupler turns off and DO is pulled high by the external pull-up resistor.

To drive DO (taking GPIO46 as an example):

```
# Export GPIO46 and configure it as an output
echo 46 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio46/direction
# Output low: the optocoupler turns on and DO is pulled low
echo 0 > /sys/class/gpio/gpio46/value
# Output high: the optocoupler turns off and DO is pulled high by the pull-up resistor
echo 1 > /sys/class/gpio/gpio46/value
```

The GPIO numbering rules may differ between kernel versions. If the nodes of GPIO47 and GPIO46 in your system differ from the example above, use `cat /sys/kernel/debug/gpio` to check the actual mapping.

## SIM Card

The SIM card slot of the AIBOX PRO KIT is used together with a 4G module to provide mobile network connectivity. **The 4G module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../aibox_img/AIBOX-PRO-KIT/sim_insert_direction.png" width="400">
</center>