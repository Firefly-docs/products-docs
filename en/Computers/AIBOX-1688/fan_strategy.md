# Fan Operation Strategy

AIBOX-1688 fan operation is divided into 4 levels:

| Current Temperature of Main Control | Fan Operation Level |
|:----------------------------------:|:-------------------:|
| ≥ 33℃                              | 1                   |
| ≥ 45℃                              | 2                   |
| ≥ 55℃                              | 3                   |
| ≥ 60℃                              | 4                   |

The higher the fan operation level, the faster the speed. If the current temperature of the main control is below 33℃, the fan is set to off by default.

Users can check the current operating temperature of the main control by executing the following command:

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

For example, the following is 45.5 ℃:

```shell
linaro@bm1688:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

Additionally, users can manually turn on the fan (write 0-4, where 0 is off and 4 is the maximum level):

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```