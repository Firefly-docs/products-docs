# RTC 使用

## 简介


Core-1109-JD4开发板采用 RK808 作为RTC(*Real Time Clock*)。RK809 集成了晶体振荡器缓冲器和实时时钟 (RTC)。缓冲器与外部 32.768kHz 晶体振荡器配合使用。借助 RTC 功能，PMIC 可提供秒/分/时/日/月/年信息、闹钟唤醒以及时间校准。RK817 提供一路 32.768kHz 时钟，具有开漏输出，默认开启并通过 I2C 接口进行控制。



## RTC驱动


驱动参考：`kernel/drivers/rtc/rtc-rk808.c`




## 接口使用

Linux 提供了三种用户空间调用接口。在 Core-1109-JD4开发板中对应的路径为：

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

设置开机时间，如设置 120 秒后开机：

```
#120秒后定时开机
echo +120 >  /sys/class/rtc/rtc0/wakealarm
# 查看开机时间
cat /sys/class/rtc/rtc0/wakealarm
#关机
reboot -p
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

