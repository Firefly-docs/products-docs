```
# cat /sys/kernel/debug/gpio
GPIOs 0-31, platform/pinctrl, gpio0:
 gpio-2   (                    |vcc3v3_3g           ) out hi
 gpio-4   (                    |bt_default_wake_host) in  lo
 gpio-5   (                    |power               ) in  hi
 gpio-9   (                    |bt_default_reset    ) out lo
 gpio-10  (                    |reset               ) out lo
 gpio-13  (                    |?                   ) out lo

GPIOs 32-63, platform/pinctrl, gpio1:
 gpio-32  (                    |vcc5v0_host         ) out hi
 gpio-34  (                    |int-n               ) in  hi
 gpio-35  (                    |vbus-5v             ) out lo
 gpio-45  (                    |pmic-hold-gpio      ) out hi
 gpio-49  (                    |vcc3v3_pcie         ) out hi
 gpio-54  (                    |mpu6500             ) out hi
 gpio-56  (                    |pmic-stby-gpio      ) out hi

GPIOs 64-95, platform/pinctrl, gpio2:
 gpio-83  (                    |bt_default_rts      ) in  hi
 gpio-90  (                    |bt_default_wake     ) in  lo
 gpio-91  (                    |?                   ) out hi

GPIOs 96-127, platform/pinctrl, gpio3:
 gpio-111 (                    |mdio-reset          ) out hi

GPIOs 128-159, platform/pinctrl, gpio4:
 gpio-149 (                    |hp-con-gpio         ) out lo
```

从读取到的信息中可以知道，内核把 GPIO 当前的状态都列出来了，以 GPIO0 组为例，gpio-2(GPIO0_A2) 作为 3G 模块的电源控制脚 (vcc3v3_3g)，输出高电平 (out hi)。


