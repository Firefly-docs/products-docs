# RTC 使用

## 简介

ROC-RK3568-PC-SE开发板采用HYM8563作为RTC(*Real Time Clock*)，HYM8563是一款低功耗CMOS实时时钟/日历芯片,它提供一个可编程的时钟输出,一个中断
输出和一个掉电检测器,所有的地址和数据都通过I2C总线接口串行传递。最大总线速度为
400Kbits/s,每次读写数据后,内嵌的字地址寄存器会自动递增

* 可计时基于 32.768kHz 晶体的秒,分,小时,星期,天,月和年
* 宽工作电压范围:1.0~5.5V
* 低休眠电流:典型值为 0.25μA(VDD =3.0V, TA =25°C)
* 内部集成振荡电容
* 漏极开路中断引脚

## RTC驱动

Android SDK中的DTS配置参考: `kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi`

```
&i2c5 {
        status = "okay";

        hym8563: hym8563@51 {
                status = "okay";
                compatible = "haoyu,hym8563";
                reg = <0x51>;
                #clock-cells = <0>;
                rtc-irq-gpio = <&gpio0 RK_PD3 IRQ_TYPE_EDGE_FALLING>;
                clock-frequency = <32768>;
        };
};
```


驱动参考：`kernel/drivers/rtc/rtc-hym8563.c`






## 接口使用

Linux 提供了三种用户空间调用接口。在 ROC-RK3568-PC-SE开发板中对应的路径为：

*    SYSFS接口：/sys/class/rtc/rtc0/
*    PROCFS接口： /proc/driver/rtc
*    IOCTL接口： /dev/rtc0

### SYSFS接口

可以直接使用 `cat` 和 `echo` 操作 `/sys/class/rtc/rtc0/` 下面的接口。

比如查看当前 RTC 的日期和时间：

```
# cat /sys/class/rtc/rtc0/date
2021-03-10
#cat /sys/class/rtc/rtc0/time
03:35:01
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
rtc_time	: 03:36:05
rtc_date	: 2021-03-10
alrm_time	: 03:37:59
alrm_date	: 2021-03-10
alarm_IRQ	: yes
alrm_pending	: no
update IRQ enabled	: no
periodic IRQ enabled	: no
periodic IRQ frequency	: 1
max user IRQ frequency	: 64
24hr		: yes
```

### IOCTL接口

可以使用 `ioctl` 控制 `/dev/rtc0`。

详细使用说明请参考文档 `kernel/Documentation/rtc.txt` 。

## FAQs

#### Q1: 开发板上电后时间不同步？

A1:  检查一下 RTC 电池是否正确接入。

