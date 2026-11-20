# RTC Usage

The AIO-1688JD4 development board has an RTC (Real Time Clock) chip `TT8563RH`, which can be used to set the system clock.

## 1. View RTC Time

```
$ sudo hwclock
2024-05-09 10:53:15.268874+0800
```

## 2. View System Time

```
$ date
Tue May  9 10:57:03 CST 2024
```

## 3. Set System Time to RTC Time

```
$ sudo hwclock -s
```

## 4. Save Current System Time to RTC

```
$ sudo hwclock -w
```