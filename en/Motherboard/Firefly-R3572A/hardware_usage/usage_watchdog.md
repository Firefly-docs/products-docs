# Watchdog Usage

## Introduction [step]

A watchdog is actually a timer that starts counting once enabled. The system or software needs to communicate with the watchdog within a specified time (commonly known as "feeding the dog") to reset the counting, repeating over and over, so as to confirm that the system and software are running normally.

If the dog is not fed within the specified time, the watchdog times out, which means the system or application is stuck in a loop or frozen. At this point the watchdog sends a reset signal to reset the main controller and escape the freeze.

This chapter mainly introduces the usage of the internal watchdog of Firefly-R3572A.

## DTS Configuration [step]

The DTS node of the watchdog of Firefly-R3572A is defined in `kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572.dtsi` (node properties are subject to the actual SDK code), as shown below:

```text
wdt: watchdog@2c0c0000 {
    compatible = "snps,dw-wdt";
    reg = <0x0 0x2c0c0000 0x0 0x100>;
    clocks = <&cru TCLK_WDT0>, <&cru PCLK_WDT0>;
    clock-names = "tclk", "pclk";
    interrupts = <GIC_SPI xxx IRQ_TYPE_LEVEL_HIGH>;
    status = "disabled";
};
```

The user first needs to enable the wdt node in the DTS file:

```text
&wdt{
    status = "okay";
};
```

## Usage [step]

The watchdog is disabled by default. Enable the related nodes in the DTS file as described above before using it.

The watchdog driver file is `kernel-6.12/drivers/watchdog/dw_wdt.c`. Two methods to use the watchdog are introduced below:

The device name of the internal watchdog is `/dev/watchdog`, and users can control the device with the `echo` command:

```shell
# Write any content (except the capital letter 'V') to enable the watchdog.
# It needs to be written once every 44 seconds (feed the dog)
echo A > /dev/watchdog

# Enable the watchdog, and the kernel feeds the dog automatically every 22 seconds
echo V > /dev/watchdog
```

You can also control the watchdog through a program. Build an executable, run it after the machine starts. The demo code is as follows:

```c
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
    int fd = open("/dev/watchdog", O_WRONLY); // start watchdog
    int ret = 0;
    if (fd == -1) {
        perror("watchdog");
        exit(EXIT_FAILURE);
    }

    ret = ioctl(fd, WDIOC_SETTIMEOUT, &timeout1); // set timeout
    if (ret < 0)
        printf("ioctl WDIOC_SETTIMEOUT failed.\n");

    ret = ioctl(fd, WDIOC_GETTIMEOUT, &timeout2); // get timeout
    if (ret < 0)
        printf("ioctl WDIOC_SETTIMEOUT failed.\n");
    printf("timeout = %d\n", timeout2);

    while (1) {
        ret = write(fd, "\0", 1); // feed the dog
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

Demo notes:

1. The internal watchdog starts counting immediately after being opened with the open function.

2. About the timeout: users can use ioctl to set and get the timeout. When the user has not set the timeout, the driver applies a default requested timeout of 30 s. Note that the timeout finally set by the driver is not necessarily the time transferred by the application or the default time initially set by the driver. There is a timeout list in the driver function, which stores 16 timeout values (the first 9 are 0). The driver will find a suitable time in the timeout list as the final timeout set by the watchdog.

The following is the detailed timeout list (taking the dw_wdt driver as an example, subject to the actual driver):

| Requested timeout | Timeout obtained via ioctl | Timeout finally set by the watchdog |
| ---- | ---- | ---- |
| timeout_request > 89 | timeout_get = timeout_request | timeout_set = 89 |
| 44 < timeout_request <= 89 | timeout_get = 89 | timeout_set = 89 |
| 22 < timeout_request <= 44 | timeout_get = 44 | timeout_set = 44 |
| 11 < timeout_request <= 22 | timeout_get = 22 | timeout_set = 22 |
| 5 < timeout_request <= 11 | timeout_get = 11 | timeout_set = 11 |
| 2 < timeout_request <= 5 | timeout_get = 5 | timeout_set = 5 |
| timeout_request = 2 | timeout_get = 2 | timeout_set = 2 |
| timeout_request = 1 | timeout_get = 1 | timeout_set = 1 |

Reference document: `docs/common/watchdog` of the SDK.

## External Watchdog [step]

Firefly-R3572A also supports an optional external watchdog controller PC9204 on the I2C bus (DTS node `pc9204: 9204@3c`, `compatible = "firefly,pc9204"`, on I2C1 at address 0x3C, enabled/disabled by the `N_WATCH_DOG` switch in the board dts), managed by the `wdt_crl` driver.

Many devices have an external hardware watchdog. Check the device files to confirm whether an external watchdog is supported. If the board has a hardware watchdog, a wdt_XXX device file should be generated under `/dev/`:

```shell
ls /dev/wdt_*
```

Enable and feed the dog by writing to the device file:

```shell
echo e > /dev/wdt_core
echo 1 > /dev/wdt_core


case '0':0.64s
case '1':2.56s
case '2':10.24s
case '3':40.96s
case 'e':enable wdt
```