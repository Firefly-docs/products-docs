# IO 
## 定义
### 模组 Core-3588SJD4-AI

* 凤凰端子引脚定义

| GND  |                 485A                 |                485A                 | GND  | CAN_H |                485A                 |
| :--: | :----------------------------------: | :---------------------------------: | :--: | :---: | :---------------------------------: |
| 12V  |                 485B                 |                485B                 | GND  | CAN_L |                485B                 |
|      | /dev/ttysWK2<br />re-de pin：GPIO122 | /dev/ttysWK1<br />re-de pin：GPIO35 |      |       | /dev/ttysWK0<br />re-de pin：GPIO34 |

* RS232 接口定义

|  上  | /dev/ttyS9 |
| :--: | :--------: |
|  下  | /dev/ttyS6 |

### 模组 Core-3576JD4

* 凤凰端子引脚定义

| GND  |                 485A                 |                485A                 | GND  | CAN_H |                485A                 |
| :--: | :----------------------------------: | :---------------------------------: | :--: | :---: | :---------------------------------: |
| 12V  |                 485B                 |                485B                 | GND  | CAN_L |                485B                 |
|      | /dev/ttysWK2<br />re-de pin：GPIO119 | /dev/ttysWK1<br />re-de pin：GPIO98 |      |       | /dev/ttysWK0<br />re-de pin：GPIO99 |

* RS232 接口定义

|  上  | /dev/ttyS8 |
| :--: | :--------: |
|  下  | /dev/ttyS3 |

## RS485 RE-DE PIN 使用

比如要操作GPIO编号是 `122`
1. 导出该 GPIO, 以 root 用户运行:
    ```
    echo 122 > /sys/class/gpio/export
    ```
2. 操作 GPIO，设定读取输入或输出指定值：
    ```
    cd /sys/class/gpio/gpio122
    
    # GPIO 设置输出模式
    echo out > direction
    # GPIO 输出低电平，RS485设置为数据接收模式
    echo 0 > value
    # GPIO 输出高电平，RS485设置为数据接收模式
    echo 1 > value
    ```

有以下注意事项：
- 如果 GPIO 已经被其他驱动申请了的话，是无法导出 (export) 成功的，也就无法进行后续的读写操作。