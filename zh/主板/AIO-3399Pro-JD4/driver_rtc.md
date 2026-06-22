# RTC 使用

## 简介

AIO-3399Pro-JD4 开发板上有 一个集成于 RK808 上的RTC(Real Time Clock)，主要功能有时钟，日历，闹钟，周期性中断，双通道 32KHz 时钟输出。

J2 接上 CR2032 纽扣电池后，可以保证板子掉电后 RTC 可以正常运行。J2 位置如下图：

![](../../../rk3399_img/AIO-3399Pro-JD4/rtc_interface.jpg)

## RTC 驱动

DTS 配置信息存放于 rk808 节点

驱动代码路径：`drivers/rtc/rtc-rk808.c`

## 接口使用

Linux 提供了三种用户空间调用接口。在 AIO-3399Pro-JD4 开发板中对应的路径为：

*    SYSFS接口：/sys/class/rtc/rtc0/
*    PROCFS接口： /proc/driver/rtc
*    IOCTL接口： /dev/rtc0

### SYSFS接口

可以直接使用 `cat` 和 `echo` 操作 `/sys/class/rtc/rtc0/` 下面的接口。

比如查看当前 RTC 的日期和时间：

```
# cat /sys/class/rtc/rtc0/date
2013-01-18
#cat /sys/class/rtc/rtc0/time
09:36:10
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

### IOCTL接口

可以使用 `ioctl` 控制 `/dev/rtc0`。

详细使用说明请参考文档 `kernel/Documentation/rtc.txt` 。

## FAQs

#### Q1: 开发板上电后时间不同步？

A1:  检查一下 RTC 电池是否正确接入。

