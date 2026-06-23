# GPIO 使用


RK3576 芯片包含 7 组 GPIO 控制器，每个控制 32 根 GPIO，与 Linux 的设备节点对应如下：

| GPIO控制器 | Linux设备节点 | GPIO 物理编号 | GPIO 逻辑编号 |
| :-------: | :---------: | :---------: | :--------: |
| #porta | /sys/class/gpio/gpiochip464 | 0到31  | 464到495 |
| #portb | /sys/class/gpio/gpiochip432 | 32到63 | 432到463 |
| #portc | /sys/class/gpio/gpiochip400 | 64到95 | 400到431 |
| #portd | /sys/class/gpio/gpiochip368 | 96到127 | 368到399 |
| #porte | /sys/class/gpio/gpiochip336 | 128到159 | 336到367 |
| #portf | /sys/class/gpio/gpiochip304 | 160到191 | 304到335 |
| #portg（RTCSYS_GPIO） | /sys/class/gpio/gpiochip272 | 0到31 | 272到303 |

其中 AIBOX-3576 支持扩展出一组 gpio 接口（I2C 扩展 GPIO）:

| GPIO控制器 | Linux设备节点 | GPIO 物理编号 | GPIO 逻辑编号 |
| :-------: | :---------: | :---------: | :--------: |
| #portx | /sys/class/gpio/gpiochip496 | 0到15 | 496到511 |

比如要操作 `porta` 的 `GPIO6`，则需要：

1. 编号转换：物理编号是 `6`， 对应的逻辑编号是 `(464 + (6 - 0)) = 470`，这里的 `0` 代表 GPIO 物理编号的开始
2. 导出该 GPIO, 以 root 用户运行:
    ```
    echo 470 >/sys/class/gpio/export
    ```
3. 操作 GPIO，设定读取输入或输出指定值：
    ```
    cd /sys/class/gpio/gpio470
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

