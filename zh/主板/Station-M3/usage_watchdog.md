
## 外部看门狗

很多设备上有外部硬件看门狗，通过查看设备文件可以确认是否支持外部看门狗。如果有硬件看门狗 /dev/ 下应该会生成 wdt_XXX 的设备文件。
```
ls /dev/wdt_*
```

通过写设备文件完成使能和喂狗。
```
echo e > /dev/wdt_core
echo 1 > /dev/wdt_core


case '0':0.64s
case '1':2.56s
case '2':10.24s
case '3':40.96s
case 'e':enable wdt
```