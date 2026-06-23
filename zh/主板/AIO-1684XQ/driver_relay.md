# RELAY 使用


AIO-1684XQ 开发板上有 2 个 RELAY 控制单元:

![](../../../bm1684_img/AIO-1684XQ/relay.png)    

每个 RELAY 单元的参数为：  

![](../../../bm1684_img/AIO-1684XQ/relay_parameter.png)
```
# RELAY0 连接状态（连接继电器的两个电路）
echo 1 >/sys/class/leds/RELAY0/brightness
# RELAY1 连接状态（连接继电器的两个电路）
echo 1 >/sys/class/leds/RELAY1/brightness

# RELAY0 断开状态（断开继电器的两个电路）
echo 0 >/sys/class/leds/RELAY0/brightness
# RELAY1 断开状态（断开继电器的两个电路）
echo 0 >/sys/class/leds/RELAY1/brightness
```
