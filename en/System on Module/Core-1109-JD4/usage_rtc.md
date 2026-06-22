# RTC

## Introduction


The Core-1109-JD4 development board uses RK808 as the RTC (*Real Time Clock*). RK809 integrates a crystal oscillator buffer and a real-time clock (RTC). The buffer is used in conjunction with an external 32.768kHz crystal oscillator. With the RTC function, the PMIC can provide seconds/minutes/hours/days/months/years information, alarm wake-up, and time calibration. RK817 provides a 32.768kHz clock with an open-drain output, which is turned on by default and controlled through the I2C interface.



## Driver RTC


Driver Reference: `kernel/drivers/rtc/rtc-rk808.c`



## Interface usage

Linux provides three user-space call interfaces. The corresponding path in the Core-1109-JD4 development board is:

*   **SYSFS Interface :** `/sys/class/rtc/rtc0/`
*   **PROCFS Interface :** `/proc/driver/rtc`
*   **IOCTL Interface :** `/dev/rtc0`

### SYSFS Interface

You can directly use the interface below `cat` and `echo` operations `/sys/class/rtc/rtc0/`.

For example, check the date and time of the current RTC:

```
# cat /sys/class/rtc/rtc0/date
2013-01-18
# cat /sys/class/rtc/rtc0/time
09:36:10
```

Set the startup time, such as starting up after 120 seconds:

```
#Start the machine regularly after 120 seconds
echo +120 >  /sys/class/rtc/rtc0/wakealarm
# View boot time
cat /sys/class/rtc/rtc0/wakealarm
#To turn it off
reboot -p
```

### PROCFS Interface

Print RTC related information:

```
# cat /proc/driver/rtc
rtc_time        : 06:53:50
rtc_date        : 2022-06-21
alrm_time       : 06:55:05
alrm_date       : 2022-06-21
alarm_IRQ       : yes
alrm_pending    : no
update IRQ enabled      : no
periodic IRQ enabled    : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

### IOCTL Interface

You can use `ioctl` to control `/dev/rtc0`.

Please refer to the document `rtc.txt` for detailed instructions.

## FAQs

#### Q1: The time is out of sync after the development board is powered on ?

**A1 :**  Check that the RTC battery is properly connected

