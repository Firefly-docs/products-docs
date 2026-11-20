# Watchdog

The external watchdog device name is `/dev/wdt_crl`, and the usage is as follows:

```shell
# Write different fields to enable the watchdog and set the time
# The numbers 0, 1, 2, 3 represent 0.64s, 2.56s, 10.24s, 40.96s respectively
# The letter e represents enabling the watchdog, and the letter d represents disabling the watchdog

# Enable and set a timeout of 10.24 seconds; it must be written to once every 10.24 seconds, and different numbers can be written at any time to change the time
echo e >/dev/wdt_crl // Enable
echo 2 >/dev/wdt_crl // Set timeout to 10 seconds
```