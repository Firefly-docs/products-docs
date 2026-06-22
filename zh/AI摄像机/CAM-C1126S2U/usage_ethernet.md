## Ethernet 使用

设备本身不带网卡，网卡是通过 USB ` RNDIS `虚拟出来的。

`CAM-C1126S2U/CAM-C1109S2U` Facial_Gate 固件默认网络配置：

* 设备 IP 地址： 172.16.110.2
* 设备默认网关： 172.16.110.1
* 设备默认 DNS： 8.8.8.8

`CAM-C1126S2U/CAM-C1109S2U` AI_UVC 固件默认网络配置：

* 设备 IP 地址： 172.16.110.6
* 设备默认网关： 172.16.110.1
* 设备默认 DNS： 8.8.8.8

如果要实现设备接入外网（适配算法时，授权可能会使用到），需要在上位机上实现多网卡流量转发。

### Linux 主机

验证是以` Ubuntu 18.04 `系统进行，其他系统可能有区别。Ubuntu 可以通过 iptables 实现流量转发。转发之前
需要设置 USB 网卡静态 IP 为` 172.16.110.5 `。

```
nmtui #进入图形界面设置 USB 网卡静态 IP

echo 1 >  /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -j MASQUERADE

iptables -F #清除掉之前所有的iptables规则
iptables -P INPUT ACCEPT #允许接收数据包
iptables -P FORWARD ACCEPT #允许发送数据包

# 在eth0 网口上 NAT ，eth0 是我 PC 对接外网的网卡，根据你自己的实际情况配置
iptables -t nat -A POSTROUTING -s 172.16.110.0/24 -o eth0 -j MASQUERADE
```

### Windows 主机

验证是以` Windows 10 `系统进行，其他系统可能有区别。Windows 10 可以通过共享网络直接实现设备上网。

* 安装驱动，连接设备开机后电脑打开设备管理器会看到 RNDIS 设备，只需要右键更新驱动就可以了。如果更新失败请尝试使用驱动精灵等第三方驱动安装工具更新安装驱动。

![](../../../rv1126_img/CAM-C1126S2U/rndis_dev.jpg)

* 设置主机自身连接外网的网卡，共享网络。

![](../../../rv1126_img/CAM-C1126S2U/rndis_eth.jpg)

* 设置 RNDIS 网卡静态 IP 为` 172.16.110.5 `。

![](../../../rv1126_img/CAM-C1126S2U/rndis_setting.jpg)