
# Watchdog 使用

## 简介

看门狗（watchdog）实际是一个定时器，启动之后会开始计时。系统或者软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如此反复下去，以此来确定系统和软件正常运行。

如果规定时间内没有喂狗，看门狗超时，说明系统或应用陷入循环、卡死，此时看门狗会发出复位信号让主控复位，脱离卡死。

本章节主要介绍 Core-1109-JD4 开发板内部看门狗的使用。


## DTS配置

Core-1109-JD4的 watchdog 的 DTS 节点在 `kernel/arch/arm/boot/dts/rv1126.dtsi` 文件中定义，如下所示：

```
wdt: watchdog@ff680000 {
	compatible = "rockchip,rv1126-wdt", "snps,dw-wdt";
	reg = <0xff680000 0x100>;
	clocks = <&cru PCLK_WDT>;
	interrupts = <GIC_SPI 32 IRQ_TYPE_LEVEL_HIGH>;
	status = "disabled";
};
```

用户首先需在 DTS 文件中打开 wdt 节点：

```
&wdt{
	status = "okay";
};
```

## 使用
watchdog 默认是关闭的，需按上述说明在 DTS 文件中打开相关节点方能使用。

watchdog 的驱动文件为 `kernel/drivers/watchdog/dw_wdt.c`。下面介绍两种方法来使用 watchdog：

内部看门狗的设备名称为`/dev/watchdog`，用户可通过 `echo` 命令来控制该设备

```
# 写入任意内容（大写字母‘V’除外），开启看门狗，每 44 秒内需要写入一次（喂狗）
echo A > /dev/watchdog

# 开启看门狗，并且内核会每隔 22 秒自动喂一次狗
echo V > /dev/watchdog
```

也可以通过程序来控制看门狗，通过交叉编译生成可执行文件，在机器启动后，push到上面运行，demo代码如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <signal.h>
#include <sys/ioctl.h>
#include <linux/types.h>
#include <linux/watchdog.h>

#define WDIOC_SETTIMEOUT        _IOWR(WATCHDOG_IOCTL_BASE, 6, int)
#define WDIOC_GETTIMEOUT        _IOR(WATCHDOG_IOCTL_BASE, 7, int)


int main(void)
{
    int timeout1 = 22;
    int timeout2;
    int fd = open("/dev/watchdog", O_WRONLY); //start watchdog
    int ret = 0;
    if (fd == -1) {
        perror("watchdog");
        exit(EXIT_FAILURE);
    }

    ret = ioctl(fd, WDIOC_SETTIMEOUT, &timeout1); //set timeout
    if (ret < 0)
        printf("ioctl WDIOC_SETTIMEOUT failed.\n"); 

    ret = ioctl(fd, WDIOC_GETTIMEOUT, &timeout2); //get timeout
    if (ret < 0)
        printf("ioctl WDIOC_SETTIMEOUT failed.\n"); 
    printf("timeout = %d\n", timeout2);

    while (1) {
        ret = write(fd, "\0", 1); //feed the dog
        if (ret != 1) {
            ret = -1;
            break;
        }
        printf("feed the dog\n");
        sleep(10);

    }
    close(fd);
    return ret;
}
```

demo说明：

1、内部看门狗在使用 open 函数打开后会立刻开始计时。

2、关于超时时间：用户可以用 ioctl 来设置超时时间和获取超时时间。当用户没有设置超时时间时，驱动会应用默认请求的超时时间为 30 s。需要说明的是驱动最终设置的超时时间并不一定是应用层传输的时间或者驱动一开始设置的默认时间。驱动函数里有一个超时时间的列表，该列表中存放了 16 个超时时间（前 9 个是 0）。驱动会在超时时间列表中找到一个合适的时间作为最终 watchdog 设置的超时时间。

以下为超时时间的详细列表：

| 请求的超时时间                |通过 ioctl 获取的超时时间              |watchdog最终设置的超时时间   |
| ----                        | ----                              | ----                     |
| timeout_request > 89        | timeout_get = timeout_request     | timeout_set =  89        |
| 44 < timeout_request <= 89  | timeout_get =  89                 | timeout_set =  89        |
| 22 < timeout_request <= 44  | timeout_get =  44                 | timeout_set =  44        |
| 11 < timeout_request <= 22  | timeout_get =  22                 | timeout_set =  22        |
| 5 < timeout_request <= 11   | timeout_get =  11                 | timeout_set =  11        |
| 2< timeout_request <= 5     | timeout_get =  5                  | timeout_set =  5         |
| timeout_request = 2         | timeout_get =  2                  | timeout_set =  2         |
| timeout_request = 1         | timeout_get =  1                  | timeout_set =  1         |

参考文档：SDK/docs/Common/WATCHDOG/
