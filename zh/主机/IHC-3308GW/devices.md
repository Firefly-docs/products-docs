## 4G

IHC-3308GW 有单wifi版本、wifi+4g版本，请确认使用的机器是拥有4G模块的。

分辨方法：检查天线数量，如果只有一个天线接口，就是单wifi版本。如果是有两个天线接口，就是wifi+4g版本。

使用的4G模块为`EC200S-CN`，对于`Buildroot`或者`Ubuntu`系统，上电之后，系统会自动进行拨号

### SIM卡连接

![](../../../rk3308_img/IHC-3308GW/sim_connect.png)

### 4G天线连接

![](../../../rk3308_img/IHC-3308GW/4g_antenna.png)

### 手动AT指令拨号联网

如果系统无法正常拨号，可以使用AT指令手动排查问题。

- 确认`EC200S-CN`模块是否正常启动，`usb0`网卡对应`EC200S-CN`模块

```bash
# ifconfig usb0
usb0      Link encap:Ethernet  HWaddr AE:0C:29:A3:9B:6D  
          inet addr:192.168.43.100  Bcast:192.168.43.255  Mask:255.255.255.0
          inet6 addr: fe80::ed5d:84b6:c27c:3825/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0
          TX packets:39 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3456 (3.3 KiB)  TX bytes:3811 (3.7 KiB)
```

- 配置串口属性

  如果是 `Ubuntu` 系统，需要进行配置

  ```bash
  # stty -F /dev/ttyUSB2 icrnl opost onlcr icanon echo echoe
  ```

- 查询模块状态

  ```bash
  # cat /dev/ttyUSB2 &
  # echo AT+QCFG="usbnet" > /dev/ttyUSB2
  ```

  如果返回`+QCFG: "usbnet",1`，即 `ECM`状态

- 模块配置为`ECM`网卡状态

  ```bash
  echo AT+QCFG="usbnet",1 > /dev/ttyUSB2
  ```

- 拨号

  ```bash
  echo AT+QNETDEVCTL=1,1,1 > /dev/ttyUSB2
  ```

- ping外网

  ![](../../../rk3308_img/IHC-3308GW/ping_usb0.png)

- 其他AT指令

断开拨号

```bash
echo AT+QNETDEVCTL=0,1,1 > /dev/ttyUSB2
```

查看天线信号的强度，返回值"0-31,99"，尽量确保信号强度在"26-31,99"

```bash
echo "AT+CSQ" > /dev/ttyUSB2 
```

查看sim卡或物联卡是否插入了，正常返回READY

```bash
echo "AT+CPIN?" > /dev/ttyUSB2
```

查看运营商，如联通CHN-UNICOM，移动"CHINA MOBILE"

```bash
echo "AT+COPS?" > /dev/ttyUSB2
```

查看sim卡的流量业务是否正常

```bash
echo "AT+CGATT?" > /dev/ttyUSB2
```

返回+CGATT: 1表示attached，+CGATT: 0表示detached，返回+CGATT: 0时请检查卡的流量业务是否正常

## Uart

扩展板上扩展了多个串口可供使用，包括 3 个 `RS485`，1个 `RS232`。

内核已默认支持上述串口功能，各串口对应的设备文件如下：

```bash
RS485_1: /dev/ttysWK0
RS485_2: /dev/ttysWK1
RS485_3: /dev/ttysWK2
RS232  : /dev/ttysWK3
```

以 RS485_1 为例：

* 连接

将 RS485_1 的 A、B 引脚分别和主机串口适配器（USB 转 485 转串口模块）的 A、B 引脚相连。

* 打开主机的串口终端

在终端打开 kermit，并设置波特率：

```bash
$ sudo kermit
C-Kermit> set line /dev/ttysWK0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

`/dev/ttyUSB0` 为主机识别到的 USB 转串口适配器的设备文件。

* 发送数据

在设备上运行如下命令：

```bash
echo "Firefly RS485 test..." > /dev/ttysWK0
```

主机中的串口终端即可接收到字符串 “Firefly RS485 test…“。

* 接收数据

首先在设备上运行下列命令：

```bash
cat /dev/ttysWK0
```

然后在主机的串口终端输入字符串 “Firefly RS485 test…”，设备端即可见到相同的字符串。

## CAN

- 连接

只需将设备的 `CANH`、`CANL` 和通讯端的 `CANH`、`CAHL` 对应连接即可。

* 发送数据

```bash
ip link set can0 down
ip link set can0 type can bitrate 250000
ip link set can0 up
cansend can0 123#1122334455667788
```

* 接收数据

```bash
ip link set can0 down
ip link set can0 type can bitrate 250000
ip link set can0 up
candump can0
```

* loopback 模式测试

```bash
ip link set can0 down
ip link set can0 type can bitrate 50000 loopback on
ip link set can0 up
candump can0 &
cansend can0 123#11223344556677
```

## DIN

网关支持一路光耦隔离接口，其中，`DI`在硬件原理图中对应于`INPUT1`，`COM`在硬件原理图中对应于`INPUT_COM`。

- 电路原理图

![](../../../rk3308_img/IHC-3308GW/gpio_input.png)

* 检测

当 `INPUT1`、`INPUT_COM` 导通时，`GPIO_INPUT1` 会检测到低电平；当 `INPUT1`、`INPUT_COM` 断开时，`GPIO_INPUT1` 会检测到高电平。

对应 `GPIO` 口如下：

```bash
GPIO_INPUT1：GPIO1_A6，38
```

检测方式如下：

```bash
# 申请 GPIO 
echo 38 > /sys/class/gpio/export
# 设置为输入
echo in > /sys/class/gpio/gpio38/direction
# 读取电平值
cat /sys/class/gpio/gpio38/value
```

## DOUT

网关支持一路继电器接口，`DO`对应于硬件原理图中的`OUTPUT1`，`COM`对应于硬件原理图中的`RELAY_COM1`。

* 电路原理图

![](../../../rk3308_img/IHC-3308GW/relay_ctl.png)

* 控制

当 `RELAY_CTL1` 输出低电平，`OUTPUT1`、`RELAY_COM1` 断开；当 `RELAY_CTL1` 输出高电平，`OUTPUT1`、`RELAY_COM1` 导通。

对应 `GPIO` 口如下：

```
RELAY_CTL1：GPIO1_B2，42
```

控制方式如下：

```bash
# 申请 GPIO 
echo 42 > /sys/class/gpio/export
# 设置为输出
echo out > /sys/class/gpio/gpio42/direction
# 设置电平值，1 / 0
echo 1 > /sys/class/gpio/gpio42/value
```

## LED

网关支持6个可自定义LED灯，分别对应的GPIO口如下：

| **L1** | GPIO2_A7 （gpio71）    |
| ------ | ---------------------- |
| **L2** | **GPIO2_A6（gpio70）** |
| **L3** | **GPIO2_B3（gpio74）** |
| **L4** | **GPIO2_B2（gpio73）** |
| **L5** | **GPIO2_B5（gpio76）** |
| **L6** | **GPIO2_B4（gpio75）** |

控制方式如下，以L1为例：

```bash
# 亮
echo 1 > /sys/class/leds/firefly\:green\:L1/brightness 
# 灭
echo 0 > /sys/class/leds/firefly\:green\:L1/brightness
```
