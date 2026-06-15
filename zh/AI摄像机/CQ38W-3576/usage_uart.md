# UART 使用

## 硬件

CQ38W-3576 使用了 `UART11` 做 `RS485`，在系统中对应 `/dev/ttyS11` 设备；扩展接口引出一个普通 `UART`，在系统中对应 `/dev/ttyS8` 设备。

接口图如下：

![](../../../rk3576_img/CQ38W-3576/usage_uart_interface.jpg)

![](../../../rk3576_img/CQ38W-3576/usage_uart_interface2.jpg)

## RS485 节点使用
```
# 关闭回显
sudo stty -F /dev/ttyS11 -echo
# 发送
sudo echo "firefly uart test..." > /dev/ttyS11   # 输入字符串
# 接收
sudo cat /dev/ttyS11
```