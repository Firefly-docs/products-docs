# RTC

## Introduction [step]

ROC-RK3568-PC development board uses HYM8563 as RTC(*Real Time Clock*), HYM8563 is a low power CMOS real-time clock/calendar chip, it provides a programmable clock output, an interrupt output and a power down detector, all addresses and data are passed serially through the I2C bus interface. The maximum bus speed is 400 Kbits/s, after each read and write data, the embedded word address register will automatically increment

- Timing can be based on 32.768kHz crystals in seconds, minutes, hours, weeks, days, months and years
- Wide working voltage range: 1.0~5.5V
- Low resting current: Typical 0.25 μA(VDD = 3.0V, TA = 25°C)
- Internal integrated oscillating capacitor
- Drain open circuit interrupt pin


## RTC驱动 [step]


## Interface usage [step]

Linux provides three user-space call interfaces. The corresponding path in the ROC-RK3568-PC development board is:

- **SYSFS Interface:** `/sys/class/rtc/rtc0/`
- **PROCFS Interface:** `/proc/driver/rtc`
- **IOCTL Interface:** `/dev/rtc0`

### SYSFS Interface

You can directly use `cat` and `echo` to operate the interfaces under `/sys/class/rtc/rtc0/`.

For example, check the date and time of the current RTC:

```console
# cat /sys/class/rtc/rtc0/date
2013-01-18
# cat /sys/class/rtc/rtc0/time
09:36:10
```

Set the startup time, such as starting up after 120 seconds:

```bash
#Start the machine regularly after 120 seconds
echo +120 >  /sys/class/rtc/rtc0/wakealarm
# View boot time
cat /sys/class/rtc/rtc0/wakealarm
#To turn it off
reboot -p
```

### PROCFS Interface

Print RTC related information:

```console
# cat /proc/driver/rtc
rtc_time        : 09:34:59
rtc_date        : 2013-01-18
alrm_time       : 08:52:45
alrm_date       : 2013-01-18
alarm_IRQ       : no
alrm_pending    : no
update IRQ enabled      : no
periodic IRQ enabled    : no
periodic IRQ frequency  : 1
max user IRQ frequency  : 64
24hr            : yes
```

### IOCTL Interface

You can use `ioctl` to control `/dev/rtc0`.

Please refer to the document `kernel/Documentation/rtc.txt` for detailed instructions.

## FAQs [step]

#### Q1: The time is out of sync after the development board is powered on ?

A1: Check that the RTC battery is properly connected.

