# RTC

## Introduction

GS1-N2 has an external RTC powered by a capacitor on the external motherboard, which ensures the RTC continues to run for a short period after power loss. In the kernel, it is represented as `rtc0`.
## Interface Usage

Linux provides three user-space interfaces for interacting with the RTC. The paths are:

*    SYSFS interface: /sys/class/rtc/rtc0/
*    PROCFS interface: /proc/driver/rtc
*    IOCTL interface: /dev/rtc0

Synchronize the current system time to the RTC.
```
hwclock -w
```

### SYSFS interface
You can directly use `cat` and `echo` to operate the interfaces under /sys/class/rtc/rtc0/.

For example, to view the current RTC date and time:
```
# cat /sys/class/rtc/rtc0/date
2024-07-10
# cat /sys/class/rtc/rtc0/time
02:36:30

```

### PROCFS interface

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

### IOCTL interface

You can use `ioctl` to control `/dev/rtc0`。

For detailed usage instructions, please refer to document `kernel-jammy-src/Documentation/admin-guide/rtc.rst` 。
