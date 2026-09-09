# Hardware Function Usage

## Login

There are two ways to login to AIBOX-Orin NX, one is via Console (Debug serial), the other is via HDMI.

### Console Login
Type-C Connects to the Console port. The login account is `nvidia`, with the password also being `nvidia`.<br>
Use the following serial port parameters:
* Baud rate: 115200
* Data bit: 8
* Stop bit: 1
* Parity check: None
* Flow control: None

### HDMI Login
When logging in via the HDMI. The login account is `nvidia`, with the password also being `nvidia`.

## Watchdog

The device names for the external watchdog is `/dev/wdt_crl`, the procedure is as follows:

```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively. 
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_crl #Enable
echo 2 >/dev/wdt_crl #Set the timeout to 10.24 seconds.
```

## RTC

### Introduction

AIBOX-Orin NX has an external RTC powered by a capacitor on the external motherboard, which ensures the RTC continues to run for a short period after power loss. In the kernel, it is represented as `rtc0`.
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