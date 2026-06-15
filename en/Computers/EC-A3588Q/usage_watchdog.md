

## External Watchdog

Many devices have external hardware watchdog. If there is a hardware watchdog, a wdt_XXX device file should be generated under /dev/.
```
ls /dev/wdt_*
```

Enabling and feeding the dog is accomplished by writing the device file.
```
echo e > /dev/wdt_core
echo 1 > /dev/wdt_core


case '0':0.64s
case '1':2.56s
case '2':10.24s
case '3':40.96s
case 'e':enable wdt
`