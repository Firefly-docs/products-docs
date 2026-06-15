# RELAY 

EC-R3576PC supports one relay output where ON corresponds to OUTPUT1 in the hardware schematic and COM corresponds to RELAY_COM1 the hardware schematic.

### Interface Diagram
![](../../../rk3576_img/CQ38W-3576/output_interface.jpg)

### schematic diagram
![](../../../rk3576_img/CQ38W-3576/output_sch.png)

### control
When GPIO3_D0_Output low level, OUTPUT1 will be disconnected with RELAY_COM1 ; When RELAY_CTL1 output high level, OUTPUT1 will be connected witch RELAY_COM1.

Since the lower layer Ext Yellow(L2) and relay are controlled by the same GPIO, controlling Ext Yellow(L2) is controlling relay output

The control mode is as follows：
```
echo 1 > /sys/class/leds/extuser/brightness //The lower monochrome led is on, the relay is worked
echo 0 > /sys/class/leds/extuser/brightness //The lower monochrome led is off, the relay isn't worked
```

[CQ38W-3576 LED wiki](usage_led.md)