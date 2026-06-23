# NETWORK Usage
AIBOX-1688 has three types of networks:
* Cellular Network (4G/5G)
* Wireless Network (WIFI/BT)
* Gigabit Ethernet Port

AIBOX-1688 has enabled these three networks by default, depending on user usage. [The location of each interface is shown in the diagram](interface_definition.html#zheng-ji-jie-kou-ding-yi).

## Cellular Network

AIBOX-1688 only has the 4G interface enabled (because the 5G network hardware conflicts with the USB3 port, requiring hardware modification to enable the 5G interface). If you make modifications and the hardware supports 5G, you can only choose one to use between the 4G and 5G interfaces.
Access steps:
* SIM card
* [EC20](https://wiki.t-firefly.com/EC20/ec20.html) /RM500U-CN
* Power on the device

In the system, the module's network card name is `wwan0`.

## Wireless Network
The wireless network interface of AIBOX-1688 is a PCIE M.2 interface. Currently, the ko files that have been compiled and built into the system are:
```
$ ls /mnt/system/ko/3rd/
$ 8852be.ko  8852ce.ko  88x2ce.ko  hci_uart.ko  rtk_btusb.ko
```
These support relevant models of WIFI and Bluetooth. Other models' modules need to be compiled using the SDK.
After the driver is loaded, the default network card name is `wlan0`.

## Gigabit Ethernet Port

The system uses `netplan` to manage configuration rules. The Ethernet port near the HDMI (eth1) has the IP address `192.168.150.1` for `ssh` login, with both the username and password as `linaro`. The Ethernet port near the USB3 socket (eth0) uses DHCP to obtain an IP address. If you want to modify the network card configuration rules, you can refer to the `/etc/netplan/` file on the device.