# RTC 使用

## 简介


AIO-1126BQ38开发板采用HYM8563作为RTC(*Real Time Clock*)，HYM8563是一款低功耗CMOS实时时钟/日历芯片,它提供一个可编程的时钟输出,一个中断输出和一个掉电检测器,所有的地址和数据都通过I2C总线接口串行传递。最大总线速度为 400Kbits/s,每次读写数据后，内嵌的字地址寄存器会自动递增。

* 可计时基于 32.768kHz 晶体的秒,分,小时,星期,天,月和年
* 宽工作电压范围:1.0~5.5V
* 低休眠电流:典型值为 0.25μA(VDD =3.0V, TA =25°C)
* 内部集成振荡电容
* 漏极开路中断引脚



## RTC驱动




## 接口使用

Linux 提供了三种用户空间调用接口。在 AIO-1126BQ38开发板中对应的路径为：

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
poweroff
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

