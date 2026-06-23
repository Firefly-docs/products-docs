# 初次使用

## 开机

AIBOX-1684 在连接电源时会自动开机，若已连接电源，请短按电源键使机器开机。

## 登录终端

登录 AIBOX-1684 的终端共有两种方式，一种是通过 Type C 串口登录，另一种是通过网络进行远程登录。

### Type C 串口登录

使用 Type C 串口登录需要准备一条 Type C 转 USB 数据线，并用其连接 AIBOX-1684 的 Type C 口 和 PC 的 USB 口。

用户可以通过 [MobaXterm](https://mobaxterm.mobatek.net/download-home-edition.html) 或 minicom 等软件连接串口，其中串口所用的波特率为115200，登录终端所使用的用户名、密码均为 `linaro`。

### 网络远程登录

使用网络远程登录前需确认 AIBOX-1684 对应网口的 IP 地址：

网口 0 （靠近 USB 口）设置了动态 IP，可用 Type C 串口登录终端后，使用 `ifconfig` 命令查看网口 0（eth0）的 IP 地址。

网口 1 （靠近 12V 电源接口）设置了静态 IP `192.168.150.1`，子网掩码 `255.255.255.0`，可以将 PC 设置成 `192.168.150.2/24` 来做初次访问。

用户可以使用以下方法快速修改 PC 的 IP 地址：

- Windows 以管理员身份打开 cmd 执行：

```bash
netsh int ipv4 set interface "以太网" dhcpstaticipcoexistence=enabled
netsh int ipv4 add address "以太网" 192.168.150.2 255.255.255.0
```

- Linux

```bash
ifconfig enp4s0:1 192.168.150.2
```

确认网口的 IP 地址后，如果能够在 PC 端成功 `ping` 通网口 IP 地址，接着就可以用 `ssh` 登录了，其中端口号为 `22`，用户名、密码同样均为 `linaro`:

```
# 192.168.150.1 需替换为对应网口的实际 IP 地址
ssh linaro@192.168.150.1
```

如需更改网口 IP 地址，请参考[网络 IP 配置](net_ip.md)章节进行操作

## 关机

注意：请先完成软/硬件关机后，再断开电源，以免损坏文件系统数据。

* 软件关机：在终端中运行 `sudo poweroff`。

* 硬件关机：长按电源键，直至工作指示灯停止闪烁。

当风扇停止运转、工作指示灯熄灭时，说明 AIBOX-1684 已完成关机，此时可安全断开电源。