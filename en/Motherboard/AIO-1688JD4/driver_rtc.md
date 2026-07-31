# RTC Usage

The AIO-1688JD4 development board has an RTC (Real Time Clock) chip `TT8563RH`, which can be used to set the system clock.

Here are common RTC operations:

```
# View RTC time
$ sudo hwclock
2024-05-09 10:53:15.268874+0800

# View system time
$ date
Tue May  9 10:57:03 CST 2024

# Set system time to RTC time
$ sudo hwclock -s

# Save current system time to RTC
$ sudo hwclock -w
```