# Watchdog
The device names for the external watchdog is `/dev/wdt_crl`, the procedure is as follows:

```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively. 
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_crl #Enable
echo 2 >/dev/wdt_crl #Set the timeout to 10.24 seconds.
```
