# Hardware Function Usage

## Login

There are two ways to log in to the AIBOX-3588S: one is terminal login through the Console (Debug Serial), and the other is login through HDMI.

### Console Login (Debug Serial)
Connect the Type-C cable to the Console port. The login account is `root`, and no `root` password is set by default.<br>
Use the following serial port parameters:
* Baud rate: 115200
* Data bits: 8
* Stop bits: 1
* Parity check: None
* Flow control: None

The location of the Console port is shown in the figure below:

<center>

<img alt="" src="../../../aibox_img/AIBOX-3588S/AIBOX-3588S-console.png" width="400">
</center>

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
When logging in through the UI, the `firefly` user is logged in automatically, and the password of the `firefly` user is also `firefly`.

## Watchdog

The external watchdog device name of the AIBOX-3588S is `/dev/wdt_crl`, and it is used as follows:

```shell
# Write different fields to enable the watchdog and set the time
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s respectively
# The letter e means enabling the watchdog, and the letter d means disabling it

# Enable it and set the timer to 10.24 seconds. It must be written to once within every 10.24 seconds. You can also write a different number at any time to change the time
echo e >/dev/wdt_crl #Enable
echo 2 >/dev/wdt_crl #Set the timeout to 10 seconds
```

## RTC

### Introduction

AIBOX-3588S has an external RTC powered by a capacitor on the baseboard, which ensures that the RTC keeps running for a short period of time after power loss. In the kernel, it is represented as `rtc0`.

### Interface Usage

Linux provides three user-space calling interfaces. The paths are:

*    SYSFS interface: /sys/class/rtc/rtc0/
*    PROCFS interface: /proc/driver/rtc
*    IOCTL interface: /dev/rtc0

Synchronize the latest system time to the RTC:
```
hwclock -w
```

#### SYSFS interface

You can directly use `cat` and `echo` to operate the interfaces under `/sys/class/rtc/rtc0/`.

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

You can use `ioctl` to control `/dev/rtc0`.

For detailed usage instructions, please refer to the document `kernel-jammy-src/Documentation/admin-guide/rtc.rst`.