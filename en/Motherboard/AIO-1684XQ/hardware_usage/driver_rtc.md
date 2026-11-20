# RTC
 

AIO-1684XQ development board is equipped with the DS1307 RTC (Real Time Clock) chip, which allows you to set the system clock using this chip.
## 1. View RTC Time

```
$ sudo hwclock
2022-08-09 10:53:15.268874+0800
```

## 2. View System Time

```
$date
Tue Aug  9 10:57:03 CST 2022
```

## 3. Set System Time to RTC Time

```
$ sudo hwclock -s
```

## 4. Save Current System Time to RTC

```
$ sudo hwclock -w
```

