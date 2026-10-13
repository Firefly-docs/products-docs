# 看门狗教程

## 简介

看门狗（watchdog）实际是一个定时器，启动之后会开始计时。系统或者软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如此反复下去，以此来确定系统和软件正常运行。

如果规定时间内没有喂狗，看门狗超时，说明系统或应用陷入循环、卡死，此时看门狗会发出复位信号让主控复位，脱离卡死。

本章节主要介绍 AIBOX-9075 开发板看门狗的使用。

## 使用

如果有硬件看门狗 /dev/ 下应该会生成 wdt_XXX 的设备文件。
```bash
ls /dev/wdt_*
```

一般为 /dev/wdt_core 或者 /dev/wdt_base

通过写设备文件完成使能和喂狗。
```bash
echo e > /dev/wdt_core
echo 1 > /dev/wdt_core

# 写入不同的值的含义
echo 0: 设置超时时间 0.64s
echo 1: 设置超时时间 2.56s
echo 2: 设置超时时间 10.24s
echo 3: 设置超时时间 40.96s
echo e: enable wdt
echo d: disable wdt
```

也可以通过程序来控制看门狗，demo 代码如下：
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <syslog.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

void main() {
    unsigned char food = '2';
    int fd_watchdog = open("/dev/wdt_core", O_WRONLY);

    if(fd_watchdog < 0) {
        printf("FAILED to open /dev/wdt_core, errno: %d, %s\n", errno, strerror(errno));
        syslog(LOG_WARNING, "FAILED to open /dev/wdt_core, errno: %d, %s", errno, strerror(errno));
        exit(1);
    }

    while (1) {
        // feed the watchdog
        ssize_t eaten = write(fd_watchdog, &food, 1);
        if(eaten != 1) {
            printf("FAILED feeding watchdog\n");
            syslog(LOG_WARNING, "FAILED feeding watchdog");
            exit(1);
        }
        sleep(8);
    }

    // close the watchdog
    food = 'd';
    write(fd_watchdog, &food, 1);
    close(fd_watchdog);
}
```