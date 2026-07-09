# Fan working strategy

AIBOX-1684 fans work in 5 levels:

| Main control current temperature | Fan working level |
|:----------:|:----------:|
| ≥ 33℃ | 1 |
| ≥ 55℃ | 2 |
| ≥ 60℃ | 3 |
| ≥ 65℃ | 4 |
| ≥ 70℃ | 5 |

The higher the working level of the fan, the faster the speed. If the current temperature of the main controller is lower than 33°C, the fan is turned off by default.

Users can view the current working temperature of the main controller by executing the following command:

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

For example the following is 45.5°C:

```shell
linaro@bm1684:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

In addition, the user can also manually turn on the fan by himself (write 0-5, 0 is off, 5 is the maximum level):

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```