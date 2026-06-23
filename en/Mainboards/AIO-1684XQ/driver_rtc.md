# RTC
 

AIO-1684XQ development board is equipped with the DS1307 RTC (Real Time Clock) chip, which allows you to set the system clock using this chip.
The following are common RTC operations:

```
# View RTC time
$ sudo hwclock
2022-08-09 10:53:15.268874+0800

# View system time
$date
Tue Aug 9 10:57:03 CST 2022

# Set system time to RTC time
$ sudo hwclock -s

# Save current system time to RTC
$ sudo hwclock -w
```

