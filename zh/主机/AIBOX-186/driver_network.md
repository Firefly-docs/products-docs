# NETWORK 使用
AIBOX-186  一共有 三种网络：
* 蜂窝网络（4G/5G）
* 无线网络（WIFI/BT）
* 千兆网口

AIBOX-186 默认使能了这三种网络，取决于用户的使用，[各接口位置如图所示](interface_definition.html#zheng-ji-jie-kou-ding-yi)  。

## 蜂窝网络

AIBOX-186 只打开了 4G 的接口（因为 5G 网络硬件上和 USB3 座子有冲突，需要修改硬件去打开 5G 接口）。假若你做了修改，在硬件支持了 5G，也只能在 4G / 5G 接口之间去选择其中一个去使用。
接入步骤：
* SIM 卡
* [EC20](https://wiki.t-firefly.com/EC20/ec20.html) /RM500U-CN
* 设备上电

在系统中，模块的网卡名字为 `wwan0`。

## 无线网络
AIBOX-186 的无线网络接口为 PCIE M.2 接口。目前系统已编译并内置好的 ko 文件有：
```
$ ls /mnt/system/ko/3rd/
$ 8852be.ko  8852ce.ko  88x2ce.ko  hci_uart.ko  rtk_btusb.ko
```
用以支持相关型号的 WIFI 以及蓝牙。其余型号的模块需要通过 SDK 进行编译生成。
驱动加载后，默认的网卡名字为 `wlan0`。

## 千兆网口

系统采取 `netplan` 去管理配置规则，默认靠近 HDMI 的网口（eth1）为 `192.168.150.1` 网段，用以 `ssh` 登录，账号密码均为` linaro` 。而靠近 USB3 座子的网口（eth0）使用 DHCP 来获取 IP。如想修改网卡配置规则，可参考设备端的 `/etc/netplan/` 文件。

