# RTC 使用
 

AIO-1684XQ 开发板上有 `DS1307` 的 RTC (Real Time Clock) 芯片，可通过该芯片去设置系统时钟。

以下是常见的 RTC 操作：

```
# 查看 RTC 时间
$ sudo hwclock
2022-08-09 10:53:15.268874+0800

# 查看系统时间
$ date
Tue Aug  9 10:57:03 CST 2022

# 设置系统时间为 RTC 时间
$ sudo hwclock -s

# 将当前系统时间保存到 RTC
$ sudo hwclock -w
```
