# RTC 教程

AIBOX-8550 搭载有 RTC (Real Time Clock) 芯片。

RTC 芯片是应用最为广泛的消费类电子产品之一，它为电子系统提供精确的时间基准，RTC 芯片大多采用精度较高的晶体振荡器作为时钟源。

RTC 芯片配有电池/超级电容，在主电源掉电时，芯片还可以工作一段时间。

## 时间读取和写入

使用 hwclock 命令可以方便的操作 RTC
```bash
# 读取当前 RTC 时间
sudo hwclock
2026-06-03 08:48:26.093819+00:00

# 直接修改 RTC 时间
sudo hwclock --set --date="20260603 15:00:00"
```

## 与系统时间进行同步

date 命令可查看当前系统时间。我们建议先修改系统时间，再将时间同步到 RTC。
```bash
# 设置系统时间
date -s "20260603 15:00:00"

# 将系统时间写入到 RTC
hwclock -w

# 将 RTC 时间设置为系统时间
hwclock -s
```

## 与网络进行同步

当设备联网，等待几分钟，后台服务会自动更新系统时间和 RTC 时间。

可以通过 timedatectl 命令查看同步状态：
```bash
firefly@ubuntu:~$ timedatectl
               Local time: Wed 2026-06-03 15:28:33 CST
           Universal time: Wed 2026-06-03 07:28:33 UTC
                 RTC time: Wed 2026-06-03 07:28:32
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: inactive
          RTC in local TZ: no
```