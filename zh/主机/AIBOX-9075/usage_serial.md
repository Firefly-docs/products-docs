# 串口教程

AIBOX-9075 有 2 个 RS485 接口。

RS485_1 对应系统中的 /dev/ttyHS1; RS485_2 对应系统中的 /dev/ttyHS3。

## 简单收发

目前 RS485 需要额外 gpio 来控制收发：
```bash
# 申请使用 695 号引脚
echo 695 > /sys/class/gpio/export

# 设置方向为输出
echo out > /sys/class/gpio/gpio695/direction

# 写入 1 表示引脚输出高电平，控制 RS485_1 进入发送模式
echo 1 > /sys/class/gpio/gpio695/value

# 发送数据
echo "RS485_TEST" > /dev/ttyHS1

# 写入 0 表示引脚输出低电平，控制 RS485_1 进入接收模式
echo 0 > /sys/class/gpio/gpio695/value

# 关闭回显并开始接收
stty -echo -F /dev/ttyHS1
cat /dev/ttyHS1

# RS485_2 同理，不过控制 gpio 是 693 号
```