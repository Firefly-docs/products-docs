# RTC
 
The AIO-1684XJD4 development board has the `HYM8563` RTC (Real Time Clock) chip, which can keep the clock running through the button battery even when the development board is powered off.
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

