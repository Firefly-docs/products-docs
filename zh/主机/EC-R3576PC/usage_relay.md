# RELAY 使用

EC-R3576PC支持一路继电器输出，其中，ON对应于硬件原理图中的OUTPUT1，COM对应于硬件原理图中的RELAY_COM1。

### 接口图
![](../../../rk3576_img/EC-R3576PC/output_interface.jpg)

### 电路原理图
![](../../../rk3576_img/EC-R3576PC/output_sch.png)

### 控制
当 GPIO3_D0_Output 输出低电平，OUTPUT1、RELAY_COM1 断开；当 RELAY_CTL1 输出高电平，OUTPUT1、RELAY_COM1 导通。

由于下层Ext Yellow(L2)与继电器为同一GPIO控制，因此控制Ext Yellow(L2)就是控制继电器输出

控制方式如下：
```
echo 1 > /sys/class/leds/extuser/brightness //下层单色灯(L2)灯亮，即继电器吸合
echo 0 > /sys/class/leds/extuser/brightness //下层单色灯(L2)灯灭，即继电器断开
```

[EC-R3576PC LED接口wiki](usage_led.md)