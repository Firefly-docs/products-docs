# RTC

## Introduction


The AIO-1126BJD4V0 development board uses the RV1126B chip's internal peripherals as the RTC (Real Time Clock). It provides year, month, day, weekday, hour, minute, and second based on a 32.768 kHz crystal oscillator. Second and hour count compensation is supported. BCD indicates time, calendar, and alarm. It supports 12 or 24-hour format, with AM and PM in 12-hour mode.

* Interrupts can be individually masked by software.
  * Alarm interrupt
  * Periodic interrupt
  * Chip power-off interrupt
  * Battery power atypical interrupt


**This RTC peripheral does not support timed sleep/wake-up.**





## Driver RTC


DTS configuration reference in Linux SDK: `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4v0.dts`

```
&rtc {
        rockchip,rtc-suspend-bypass;
        status = "okay";
};
```

Driver Reference: `kernel-6.1/drivers/rtc/rtc-rockchip.c`




## Interface usage

Linux provides three user-space call interfaces. The corresponding path in the AIO-1126BJD4V0 development board is:

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

