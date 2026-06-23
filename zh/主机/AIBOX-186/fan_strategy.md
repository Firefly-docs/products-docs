# 风扇工作策略

AIBOX-186 风扇工作分为 4 个等级：

| 主控当前温度 | 风扇工作等级 |
|:----------:|:----------:|
| ≥ 33℃ | 1 |
| ≥ 45℃ | 2 |
| ≥ 55℃ | 3 |
| ≥ 60℃ | 4 |

风扇工作等级越高，转速越快，如果主控当前温度低于 33℃，风扇默认是关闭状态。

用户可通过执行以下命令查看当前主控的工作温度：

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

例如以下是 45.5 ℃：

```shell
linaro@bm1688:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

另外，用户也可以自己手动打开风扇（写入 0-4， 0 是关闭， 4 是最大等级）：

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```