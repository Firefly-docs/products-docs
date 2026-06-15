## USB 以太网

USB 以太网，主要实现的是将设备的 OTG 接口做外设模式，模拟成一个网络接口，然后主机通过 USB 连接设备并通过设备访问互联网。以下是基于 Firefly-RK3399 板卡进行的具体操作。

操作环境：

* Ubuntu 系统的 PC 机
* Firefly-RK3399 板卡


### 内核设置

在内核目录下，打开内核配置选项菜单：

```
make firefly_linux_defconfig
make menuconfig
```

进入内核配置菜单后依次选择：`Device Drivers` -> `USB Support` -> `USB Gadget Support`。

将 `USB Gadget Driver` 设置成编译成模块，然后可以在下边找到 `Ethernet Gadget (with CDC Ethernet support)` 选项，同样选择编译成模块。同时要选择上 `RNDIS support`。

```
<M>   USB Gadget Drivers
<M>     USB functions configurable through configfs
<M>     Ethernet Gadget (with CDC Ethernet support)
[*]       RNDIS support (NEW)
```

接着在 `kernel` 目录下编译内核：

```
make rk3399-firefly.img -j12
```

编译完成后将内核烧录到设备中，烧录过程请用户参考相应板卡维基的**升级固件**页面。然后将内核目录下生成的下列模块复制到设备中：

* drivers/usb/gadget/function/u_ether.ko
* drivers/usb/gadget/function/usb_f_ecm_subset.ko
* drivers/usb/gadget/function/usb_f_ecm.ko
* drivers/usb/gadget/function/usb_f_rndis.ko
* drivers/usb/gadget/function/usb_f_eem.ko
* drivers/usb/gadget/legacy/g_ether.ko
* drivers/usb/gadget/libcomposite.ko

然后在设备上，依次加载上述模块：

```
insmod libcomposite.ko
insmod u_ether.ko
insmod usb_f_ecm_subset.ko
insmod usb_f_rndis.ko
insmod usb_f_ecm.ko
insmod usb_f_eem.ko
insmod g_ether.ko
```

**注意：** 要先加载 `libcomposite.ko` 和 `u_ether.ko`，后面的模块才可以加载进去。

### IP 地址设置

用数据线连接 PC 机和设备的 OTG 接口，在 PC 机中执行 `lsusb` 命令可以查看到 USB 以太网设备，即说明连接成功。

```
firefly@Desktop:~$ lsusb
Bus 002 Device 003: ID 09da:5814 A4Tech Co., Ltd.
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 005: ID 04f2:b2ea Chicony Electronics Co., Ltd Integrated Camera [ThinkPad]
Bus 001 Device 004: ID 0a5c:21e6 Broadcom Corp. BCM20702 Bluetooth 4.0 [ThinkPad]
Bus 001 Device 003: ID 147e:1002 Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

输出信息中 `ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget` 即为 USB 网卡设备。

设备插入网线，使设备可以连接外网。

* 在设备中 IP 的设置：

输入执行 `ifconfig -a` 命令，可以查看到以下信息：

```bash
root@firefly:~# ifconfig -a

# eth0 是有线网卡
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 168.168.100.48  netmask 255.255.0.0  broadcast 168.168.255.255
        inet6 fe80::1351:ae2f:442e:e436  prefixlen 64  scopeid 0x20<link>
        ether 8a:4f:c3:77:94:ac  txqueuelen 1000  (Ethernet)
        RX packets 9759  bytes 897943 (897.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 236  bytes 35172 (35.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 42  base 0x8000

        ...

# usb0 是虚拟的 usb 网卡
usb0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 4a:81:b1:34:d2:ad  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

然后给 usb0 网卡自定义一个适当的 IP：

**注意要设置 usb0 的 IP 和有线网卡 eth0 的 IP 不在同一网段！！**

```
ifconfig usb0 192.168.1.101
```

* 在 PC 机中 IP 的设置：

```bash
# 先查看 USB 虚拟网卡
firefly@Desktop:~$ ifconfig
enp0s20u2i1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.2.90  netmask 255.255.255.0  broadcast 192.168.2.255
        inet6 fe80::871c:b87e:1327:7fd4  prefixlen 64  scopeid 0x20<link>
        ether 46:fe:6e:97:ee:a6  txqueuelen 1000  (以太网)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1  bytes 54 (54.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...


# 设置 USB 网卡的 IP 地址和设备的 usb0 的 IP 地址在同一网段
firefly@Desktop:~$ sudo ifconfig enp0s20u2i1 192.168.1.100

#设置默认网关：要设置为设备 usb0 的 ip 地址，因为后面要通过 usb0 来进行流量的转发
firefly@Desktop:~$ sudo route add default gw 192.168.1.101
```

设置完设备和 PC 机的 IP 后，互相是可以 `ping` 通的，PC 机也可以用 `ssh` 命令登录到设备。

### 网络共享实现 PC 机上网

在设备上：首先打开 IPv4 的转发功能：

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

如果希望每次重启设备后都自动打开转发功能，请直接修改 `/etc/sysctl.conf` 文件的 `net.ipv4.ip_forward` 值为1。修改文件参数后要执行 `sysctl -p` 命令重新载入 `/etc/sysctl.conf` 文件，使 IPv4 转发功能生效。

添加规则进行流量转发：

```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 168.168.100.48
```

现在 PC 主机就可以访问网络了。如果 PC 机可以 ping 通设备的 usb0 和 eth0，但是无法上外网，则需要修改 PC 机的 DNS，在 `/etc/resolv.conf` 中做以下添加：

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

配置过程中要注意以下几点：

* 对应好上述步骤中自己设备上的各个 IP 地址,注意设备上的 USB 虚拟网卡 IP 和有线网络 IP 不在同一网段;
* PC 机虚拟 USB 网卡的网关要设置为设备虚拟 USB 网卡的 IP 地址;
* 设备上的虚拟网卡 IP 地址、IP 转发功能、流量转发规则等设置会在设备重启后恢复，用户可以自行查找资料如何开机自动设置这些配置。