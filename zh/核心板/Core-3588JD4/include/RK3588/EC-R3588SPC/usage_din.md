# DIN 使用

EC-R3588SPC支持一路光耦隔离输入，其中，IN在硬件原理图中对应于INPUT1，G在硬件原理图中对应于INPUT_COM。

### 接口图
![](../../../rk3588_img/Core-3588JD4/input_interface.jpg)

### 电路原理图
![](../../../rk3588_img/Core-3588JD4/input_sch.jpg)

### 检测
原理图中的`INPUT1`对应丝印的`IN`，`INPUT_COM`对应`G`。当 `INPUT1(IN)`与`INPUT_COM(G)`导通时，`GPIO3_D2_INPUT` 会检测到低电平；当 `INPUT1(IN)`与`INPUT_COM(G)` 断开时，`GPIO3_D2_INPUT` 会检测到高电平。

* 例1：`INPUT(IN)`接5V，那么就是检测`INPUT_COM(G)`的输入状态，当`INPUT_COM(G)`为低，主控检测到的`GPIO3_D2_INPUT`为低；

* 例2：`INPUT(IN)`想接入24V的话，需要串联个电阻（阻值为3.9K至4.7K之间）后再接入到`INPUT(IN)`以防止烧坏光耦隔离芯片，逻辑与例1一致；

<font color=#FF0000>注意：强烈建议采用例1的方案，如有其他配置想法，请根据实际电路原理图来配置`INPUT(IN)`与`INPUT_COM(G)`，以防烧坏光耦</font>

检测方式如下：
```
# 申请 GPIO 
echo 122 > /sys/class/gpio/export
# 设置为输入
echo in > /sys/class/gpio/gpio122/direction
# 读取电平值
cat /sys/class/gpio/gpio122/value
```