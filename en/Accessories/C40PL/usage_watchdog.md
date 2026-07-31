
# Watchdog

## Introduction

Watchdog is actually a timer that will start counting once the board is powered up. System or software needs to repeatedly communicate with it and reset the countdown during a specific period to detect and recover from malfunctions.This process is typically called feeding.

If watchdog is not fed in time or there is any timeouts,  system or application being trapped in a cycle or being stuck occur. At this time, watchdog will send a signal to reset the SOC in order to pull the system or application out of the current situation.

C40PL development board includes an integrated, on-chip watchdog. This section will show you how to manage it.

## DTS configuration

The watchdog node of C40PL defined in `kernel/arch/arm/boot/dts/rv1126.dtsi`, as showm below:

```
wdt: watchdog@ff680000 {
	compatible = "rockchip,rv1126-wdt", "snps,dw-wdt";
	reg = <0xff680000 0x100>;
	clocks = <&cru PCLK_WDT>;
	interrupts = <GIC_SPI 32 IRQ_TYPE_LEVEL_HIGH>;
	status = "disabled";
};
```

Firstly, user needs to open the WDT node in the DTS file:

```
&wdt{
    status = "okay";
};
```

## usage

The watchdog is disabled by default. You need to open the related node in the DTS file as described above.

The driver file of the watchdog is `kernel/drivers/watchdog/dw_wdt.c`. Here are two ways to use watchdog:

The device name of internal watchdog is `/dev/watchdog`. You can control this device by running the `echo` command.

```
# Writing any character except capital V will open the watchdog, and write once every 44 seconds.
echo A > /dev/watchdog

# Turning on the watchdog, and the kernel will every 22 seconds feed the dog automatically.
echo V > /dev/watchdog
```

You can also control the watchdog through the program. Compiling the executable file through Android.mk, and running the executable file after the machine is started. The demo code is as follows:

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

The instruction of demo:

1、The internal watchdog will start timing immediately after it is opened with the open function.

2、About timeout:
The user can set the timeout period and obtain the timeout period by the ioctl function. When the user does not set the timeout, the driver will apply the default request timeout of 30 seconds. It should be noted that the timeout time finally set by the driver is not necessarily the transmission time of the application layer or the default time set by the driver at the beginning. There is a list of timeout times in the driver function, which stores 16 timeout times and the first nine timeout times are zero. The driver will find a suitable time in the timeout list as the timeout set by the watchdog.

The following is a detailed list of timeout times:

| The timeout period of the request  |Timeout obtained by the ioctl   |Final timeout set by watchdog   |
| ----                               | ----                           | ----                           |
| timeout_request > 89               | timeout_get = timeout_request  | timeout_set =  89              |
| 44 < timeout_request <= 89         | timeout_get =  89              | timeout_set =  89              |
| 22 < timeout_request <= 44         | timeout_get =  44              | timeout_set =  44              |
| 11 < timeout_request <= 22         | timeout_get =  22              | timeout_set =  22              |
| 5 < timeout_request <= 11          | timeout_get =  11              | timeout_set =  11              |
| 2< timeout_request <= 5            | timeout_get =  5               | timeout_set =  5               |
| timeout_request = 2                | timeout_get =  2               | timeout_set =  2               |
| timeout_request = 1                | timeout_get =  1               | timeout_set =  1               |

Reference documents:

linux :SDK/docs/common/watchdog
