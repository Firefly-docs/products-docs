# RTC 使用

## 简介 [step]

Firefly-R3572A 板载 HYM8563 实时时钟芯片（RTC，*Real Time Clock*）。HYM8563 是一款低功耗 CMOS 实时时钟/日历芯片，提供可编程时钟输出、中断输出与掉电检测，所有地址和数据通过 I2C 总线串行传输（最大 400 kbit/s）：

* 基于 32.768 kHz 晶振计时（秒、分、小时、星期、日、月、年）
* 宽工作电压范围 1.0～5.5 V
* 低休眠电流（典型 0.25 μA，VDD=3.0 V，TA=25 °C）
* 内部集成振荡电容，漏极开路中断引脚

## RTC 驱动 [step]

Firefly-R3572A 的 HYM8563 挂在 I2C1 总线、地址 0x51，DTS 节点位于 `kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-r3572a.dtsi`（由主 DTS 的 `N_RTC` 开关控制是否启用）：

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

驱动源码为 `kernel-6.12/drivers/rtc/rtc-hym8563.c`。节点名、中断 GPIO、`wakeup-source` 等属性以你手中 SDK 的代码为准。

## 接口使用 [step]

Linux 提供三种用户空间接口访问 RTC：

* SYSFS 接口：`/sys/class/rtc/rtc0/`
* PROCFS 接口：`/proc/driver/rtc`
* IOCTL 接口：`/dev/rtc0`

### SYSFS 接口

查看当前 RTC 日期与时间：

```bash
cat /sys/class/rtc/rtc0/date
cat /sys/class/rtc/rtc0/time
```

设置定时开机（如 120 秒后开机）：

```bash
echo +120 > /sys/class/rtc/rtc0/wakealarm
cat /sys/class/rtc/rtc0/wakealarm     # 查看设定的开机时间
reboot -p                              # 关机，等待定时开机
```

### PROCFS 接口

```bash
cat /proc/driver/rtc
```

输出示例：

```text
rtc_time        : 06:53:50
rtc_date        : 2022-06-21
alrm_time       : 06:55:05
alrm_date       : 2022-06-21
alarm_IRQ       : yes
alrm_pending    : no
24hr            : yes
```

### IOCTL 接口

可通过 `ioctl` 操作 `/dev/rtc0`，详细说明参考 SDK 文档 `kernel-6.12/Documentation/admin-guide/rtc.rst`。

## FAQs [step]

### Q1：开发板上电后时间不同步？

A1：检查 RTC 电池是否正确接入；若使用 `wakealarm` 定时开机，确认 `wakeup-source` 属性已使能且系统时间已正确写入 RTC。

### Q2：系统时间与 RTC 时间如何同步？

A2：可通过 `hwclock` 在两者间同步，例如 `hwclock -w` 将系统时间写入 RTC，`hwclock -s` 用 RTC 时间设置系统时间。
