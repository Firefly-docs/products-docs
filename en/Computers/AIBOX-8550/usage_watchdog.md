# Watchdog Usage

## Introduction

Watchdog is actually a timer that will start counting once the board is powered up. System or software needs to repeatedly communicate with it and reset the countdown during a specific period to detect and recover from malfunctions.This process is typically called feeding.

If watchdog is not fed in time or there is any timeouts,  system or application being trapped in a cycle or being stuck occur. At this time, watchdog will send a signal to reset the SOC in order to pull the system or application out of the current situation.

AIBOX-8550 development board includes a watchdog. This section will show you how to manage it.

## Usage

If there is a hardware watchdog, a wdt_XXX device file should be generated under /dev/.
```bash
ls /dev/wdt_*
```

It will be /dev/wdt_core or /dev/wdt_base

Enabling and feeding the dog is accomplished by writing the device file.
```bash
echo e > /dev/wdt_core
echo 1 > /dev/wdt_core

# echo different value means
echo 0: set timeout 0.64s
echo 1: set timeout 2.56s
echo 2: set timeout 10.24s
echo 3: set timeout 40.96s
echo e: enable wdt
echo d: disable wdt
```

You can also control the watchdog through the program. The demo code is as follows:
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