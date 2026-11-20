# RTC 使用

AIO-186JD4 开发板上有 `TT8563RH` 的 RTC (Real Time Clock) 芯片，可通过该芯片去设置系统时钟。

## 1. 查看 RTC 时间

```
$ sudo hwclock
2024-05-09 10:53:15.268874+0800
```

## 2. 查看系统时间

```
$ date
```

## 3. 设置系统时间为 RTC 时间

```
$ sudo hwclock -s
```

## 4. 将当前系统时间保存到 RTC

```
$ sudo hwclock -w
```
