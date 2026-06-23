# GPIO 使用

BM1684X 芯片包含 3 组 GPIO 控制器，每个控制 32 根 GPIO，与 Linux 的设备节点对应如下：

| GPIO控制器 | Linux设备节点 | GPIO 物理编号 | GPIO 逻辑编号 |
| :-------: | :---------: | :---------: | :--------: |
| #0 | /sys/class/gpio/gpiochip480 | 0到31  | 480到511 |
| #1 | /sys/class/gpio/gpiochip448 | 32到63 | 448到479 |
| #2 | /sys/class/gpio/gpiochip416 | 64到95 | 416到447 |

比如要操作 `GPIO6`，则需要：

1. 编号转换：物理编号是 `6`， 对应的逻辑编号是 `(480 + (6 - 0)) = 486`，这里的 `0` 代表 GPIO 物理编号的开始
2. 导出该 GPIO, 以 root 用户运行:
    ```
    echo 486 >/sys/class/gpio/export
    ```
3. 操作 GPIO，设定读取输入或输出指定值：
    ```
    cd /sys/class/gpio/gpio486
    # 设置输入模式
    echo in >direction
    cat value
    
    # 设置输出模式
    echo out >direction
    # 输出低电平
    echo 0 >value
    # 输出高电平
    echo 1 >value
    ```

有以下注意事项：
- 如果 GPIO 已经被其他驱动申请了的话，是无法导出 (export) 成功的，也就无法进行后续的读写操作。

 

同时 AIO-1684XQ  通过双层的接线端子预先注册了 4 个 GPIO 可供用户直接操作。
![](../../../bm1684_img/AIO-1684XQ/gpio.png)

例如操作  `GPIO5` ： 
```
# 拉高
echo 1 >/sys/class/leds/GPIO5/brightness
# 拉低
echo 0 >/sys/class/leds/GPIO5/brightness
```


