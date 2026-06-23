# RTC
 
The AIBOX-K3 development board has the `HYM8563` RTC (Real Time Clock) chip, which can keep the clock running through the button battery even when the development board is powered off.
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

