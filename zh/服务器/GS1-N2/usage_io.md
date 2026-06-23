# IO 
## 定义
### 主模组 Core-3588JD4

| 名称 | GPIO编号 | IO类型 |
| --- | --- | --- |
| IO1 | 36 | 输入  |
| IO2 | 102 | 输入  |
| IO3 | 44 | 输入  |
| IO4 | 38 | 输入  |
| IO5 | 137 | 输出(继电器) |

### 主模组 Core-3576JD4

| 名称 | GPIO编号 | IO类型 |
| --- | --- | --- |
| IO1 | 127 | 输入  |
| IO2 | 125 | 输入  |
| IO3 | 129 | 输入  |
| IO4 | 126 | 输入  |
| IO5 | 128 | 输出(继电器) |

### 主模组 Jetson Orin Nano/Jetson Orin NX

| 名称 | GPIO编号 | IO类型 |
| --- | --- | --- |
| IO1 | 397 | 输入  |
| IO2 | 391 | 输入  |
| IO3 | 453 | 输入  |
| IO4 | 486 | 输入  |
| IO5 | 441 | 输出(继电器) |

## 使用 
比如要操作GPIO编号是 470
1. 导出该 GPIO, 以 root 用户运行:
    ```
    echo 470 > /sys/class/gpio/export
    ```
2. 操作 GPIO，设定读取输入或输出指定值：
    ```
    cd /sys/class/gpio/gpio470
    # 设置输入模式
    echo in > direction
    cat value
    
    # 设置输出模式
    echo out > direction
    # 输出低电平
    echo 0 > value
    # 输出高电平
    echo 1 > value
    ```

有以下注意事项：
- 如果 GPIO 已经被其他驱动申请了的话，是无法导出 (export) 成功的，也就无法进行后续的读写操作。
