

```
# cat /sys/kernel/debug/gpio
GPIOs 0-31, platform/pinctrl, gpio0:
 gpio-1   (                    |?                   ) out hi
 gpio-4   (                    |sysfs               ) out hi
 gpio-5   (                    |bt_default_wake_host) in  hi
 gpio-10  (                    |sysfs               ) out hi
 gpio-11  (                    |sysfs               ) out hi

GPIOs 32-63, platform/pinctrl, gpio1:
 gpio-32  (                    |sysfs               ) out hi
 gpio-33  (                    |vcc3v3_3g           ) out hi
 gpio-35  (                    |sysfs               ) out hi
 gpio-36  (                    |sysfs               ) out lo
 gpio-45  (                    |?                   ) out lo
 gpio-46  (                    |vsel                ) out lo
 gpio-49  (                    |vsel                ) out lo
 gpio-54  (                    |sysfs               ) out hi
 gpio-55  (                    |sysfs               ) out hi
 gpio-56  (                    |sysfs               ) out hi

GPIOs 64-95, platform/pinctrl, gpio2:
 gpio-70  (                    |reset-gpio          ) out hi
 gpio-72  (                    |irq-gpio            ) in  hi
 gpio-76  (                    |cs-gpio             ) out hi
 gpio-83  (                    |bt_default_rts      ) in  hi
 gpio-90  (                    |bt_default_wake     ) in  hi
 gpio-91  (                    |reset               ) out hi
 gpio-92  (                    |bt_default_reset    ) out lo

GPIOs 96-127, platform/pinctrl, gpio3:
 gpio-111 (                    |mdio-reset          ) out hi

GPIOs 128-159, platform/pinctrl, gpio4:
 gpio-149 (                    |vbus-gpio           ) out hi
 gpio-154 (                    |vcc5v0_host         ) out hi

GPIOs 511-511, platform/rk805-pinctrl, rk817-gpio, can sleep:
```

从读取到的信息中可以知道，内核把 GPIO 当前的状态都列出来了，以 GPIO1 组为例，gpio-33(GPIO1_A1) 作为 3G 模块的电源控制脚 (vcc3v3_3g)，输出高电平 (out hi)。

