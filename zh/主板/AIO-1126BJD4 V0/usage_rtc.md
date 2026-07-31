# RTC 使用

## 简介


AIO-1126BJD4V0 开发板采用 RV1126B 芯片内部的外设作为RTC(*Real Time Clock*)。基于 32.768 kHz 晶体振荡器提供年、月、日、星期、小时、分钟和秒。 支持秒和小时计数补偿。BCD表示时间、日历和闹钟。12 或 24 小时制，AM 和 PM 为 12 小时模式。

* 中断可单独通过软件屏蔽
  * 报警中断
  * 周期性中断
  * 芯片断电中断
  * 电池电源非典型中断

**该 RTC 外设不支持定时休眠唤醒。**



## RTC驱动


Linux SDK 中的 DTS 配置参考: `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4v0.dts`

```
&rtc {
        rockchip,rtc-suspend-bypass;
        status = "okay";
};
```

驱动参考：`kernel-6.1/drivers/rtc/rtc-rockchip.c`




## 接口使用

Linux 提供了三种用户空间调用接口。在 AIO-1126BJD4V0开发板中对应的路径为：

*    SYSFS接口：/sys/class/rtc/rtc0/
*    PROCFS接口： /proc/driver/rtc
*    IOCTL接口： /dev/rtc0

### SYSFS接口

可以直接使用 `cat` 和 `echo` 操作 `/sys/class/rtc/rtc0/` 下面的接口。

比如查看当前 RTC 的日期和时间：

```
# cat /sys/class/rtc/rtc0/date 
2022-06-21
# cat /sys/class/rtc/rtc0/time 
06:52:08
```


### PROCFS 接口

打印 RTC 相关的信息：

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

### IOCTL接口

可以使用 `ioctl` 控制 `/dev/rtc0`。

详细使用说明请参考文档 `kernel/Documentation/admin-guide/rtc.rst` 。

## FAQs

#### Q1: 开发板上电后时间不同步？

A1:  检查一下 RTC 电池是否正确接入。

