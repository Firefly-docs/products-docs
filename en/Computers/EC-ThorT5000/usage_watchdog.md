
# Watchdog

## Introduction

Watchdog is actually a timer that will start counting once the board is powered up. System or software needs to repeatedly communicate with it and reset the countdown during a specific period to detect and recover from malfunctions.This process is typically called feeding.

If watchdog is not fed in time or there is any timeouts,  system or application being trapped in a cycle or being stuck occur. At this time, watchdog will send a signal to reset the SOC in order to pull the system or application out of the current situation.

This section will show you how to use watchdog on EC-ThorT5000. 

## External Watchdog

EC-ThorT5000 also has an external watchdog,device node in system is `/dev/wdt_crl`.

Usage:
```bash
# Enable the wdt
echo e > /dev/wdt_crl

# Set the timeout, there're 4 available values:
echo 0 > /dev/wdt_crl # 0.64 Sec
echo 1 > /dev/wdt_crl # 2.56 Sec
echo 2 > /dev/wdt_crl # 10.24 Sec
echo 3 > /dev/wdt_crl # 40.96 Sec

# Disable the wdt
echo d > /dev/wdt_crl
```

