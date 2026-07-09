
# Watchdog 使用

## 简介

看门狗（watchdog）实际是一个定时器，启动之后会开始计时。系统或者软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如此反复下去，以此来确定系统和软件正常运行。
如果规定时间内没有喂狗，看门狗超时，说明系统或应用陷入循环、卡死，此时看门狗会发出复位信号让主控复位，脱离卡死。
本章节主要介绍 EC-AGXOrin 开发板看门狗的使用。

## 外部看门狗

EC-AGXOrin 有 1 个外部看门狗，对应 `/dev/wdt_crl` 。

使用方法：
```bash
# 开启看门狗
echo e > /dev/wdt_crl

# 设置超时时间，共 4 个挡位
echo 0 > /dev/wdt_crl # 0.64 秒
echo 1 > /dev/wdt_crl # 2.56 秒
echo 2 > /dev/wdt_crl # 10.24 秒
echo 3 > /dev/wdt_crl # 40.96 秒

# 关闭看门狗
echo d > /dev/wdt_crl
```

