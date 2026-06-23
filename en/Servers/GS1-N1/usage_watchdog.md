# Watchdog
## Main Module - Core-3588SJD4-AI

### Main Module

The device names for the external watchdogs is `/dev/wdt_core`. For example, using `/dev/wdt_core`, the procedure is as follows:
```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively.
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_core //Enable
echo 2 >/dev/wdt_core //Set the timeout to 10.24 seconds.
```

### Main Board
The device names for the external watchdogs is `/dev/wdt_base`. For example, using `/dev/wdt_base`, the procedure is as follows:
```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively.
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_base //Enable
echo 2 >/dev/wdt_base //Set the timeout to 10.24 seconds.
```

## Main Module - Core-3576JD4
### Main Module
The device names for the external watchdogs is `/dev/wdt_core`. For example, using `/dev/wdt_core`, the procedure is as follows:
```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively.
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_core //Enable
echo 2 >/dev/wdt_core //Set the timeout to 10.24 seconds.
```

### Main Board
The device names for the external watchdogs is `/dev/wdt_base`. For example, using `/dev/wdt_base`, the procedure is as follows:
```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively.
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_base //Enable
echo 2 >/dev/wdt_base //Set the timeout to 10.24 seconds.
```

## Main Module - Jetson Orin Nano / Jetson Orin NX
### Main Board
The device names for the external watchdog is `/dev/wdt_crl`, the procedure is as follows:

```shell
# Write to different fields to enable the watchdog and set the time.
# The numbers 0, 1, 2, and 3 represent 0.64s, 2.56s, 10.24s, and 40.96s, respectively. 
# The letter 'e' indicates enabling the watchdog, while the letter 'd' indicates disabling it.

# Enable and set the timer for 10.24 seconds. You need to write to it once every 10.24 seconds. You can also write different numbers at any time to change the timing.
echo e >/dev/wdt_crl //Enable
echo 2 >/dev/wdt_crl //Set the timeout to 10.24 seconds.
```
