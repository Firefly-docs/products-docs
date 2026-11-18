# RTC Usage

## Introduction [step]

Firefly-R3572A has an on-board HYM8563 real-time clock (RTC) chip. HYM8563 is a low-power CMOS RTC/calendar chip that provides a programmable clock output, an interrupt output and a power-fail detector. All addresses and data are transferred serially over the I2C bus (up to 400 kbit/s):

* Timekeeping based on a 32.768 kHz crystal (seconds, minutes, hours, weekday, day, month, year)
* Wide operating voltage range: 1.0–5.5 V
* Low standby current (typically 0.25 μA, VDD=3.0 V, TA=25 °C)
* Integrated oscillation capacitor and open-drain interrupt pin

## RTC Driver [step]

The HYM8563 of Firefly-R3572A is on the I2C1 bus at address 0x51, with its DTS node in `kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-r3572a.dtsi` (enabled/disabled by the `N_RTC` switch in the board dts):

```text
&i2c1 {
    status = "okay";
    pinctrl-0 = <&i2c1m1_xfer_pins>;

    hym8563: hym8563@51 {
        compatible = "haoyu,hym8563";
        reg = <0x51>;
        #clock-cells = <0>;
        clock-frequency = <32768>;
        clock-output-names = "hym8563";
        pinctrl-names = "default";
        pinctrl-0 = <&hym8563_int>;
        interrupt-parent = <&gpio0>;
        interrupts = <RK_PC2 IRQ_TYPE_LEVEL_LOW>;
        wakeup-source;
    };
};
```

The driver source is `kernel-6.12/drivers/rtc/rtc-hym8563.c`. The node name, interrupt GPIO and `wakeup-source` property are subject to your SDK code.

## Interface Usage [step]

Linux provides three user-space interfaces to access the RTC:

* SYSFS interface: `/sys/class/rtc/rtc0/`
* PROCFS interface: `/proc/driver/rtc`
* IOCTL interface: `/dev/rtc0`

### SYSFS Interface

Check the current RTC date and time:

```bash
cat /sys/class/rtc/rtc0/date
cat /sys/class/rtc/rtc0/time
```

Set a timed power-on (e.g. 120 seconds later):

```bash
echo +120 > /sys/class/rtc/rtc0/wakealarm
cat /sys/class/rtc/rtc0/wakealarm     # show the scheduled power-on time
reboot -p                              # power off and wait for the timed power-on
```

### PROCFS Interface

```bash
cat /proc/driver/rtc
```

Example output:

```text
rtc_time        : 06:53:50
rtc_date        : 2022-06-21
alrm_time       : 06:55:05
alrm_date       : 2022-06-21
alarm_IRQ       : yes
alrm_pending    : no
24hr            : yes
```

### IOCTL Interface

The `/dev/rtc0` device can be controlled via `ioctl`. For details, refer to `kernel-6.12/Documentation/admin-guide/rtc.rst` in the SDK.

## FAQs [step]

### Q1: The time is not synchronized after power-on?

A1: Check whether the RTC battery is properly connected. If using `wakealarm` for timed power-on, make sure the `wakeup-source` property is enabled and the system time has been written to the RTC.

### Q2: How to synchronize the system time with the RTC?

A2: Use `hwclock`, for example `hwclock -w` writes the system time to the RTC, and `hwclock -s` sets the system time from the RTC.
