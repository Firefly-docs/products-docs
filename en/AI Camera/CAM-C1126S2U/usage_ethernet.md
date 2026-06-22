## Ethernet

The device itself does not have a network card, which is virtualized through USB `RNDIS`.

`CAM-C1126S2U/CAM-C1109S2U` Facial_Gate firmware default network configuration:

* Device IP address: 172.16.110.2
* Device default gateway: 172.16.110.1
* Device default DNS: 8.8.8.8

`CAM-C1126S2U/CAM-C1109S2U` AI_UVC firmware default network configuration:

* Device IP address: 172.16.110.6
* Device default gateway: 172.16.110.1
* Device default DNS: 8.8.8.8

If you want to connect the device to the external network (authorization may be used when adapting the algorithm), you need to implement multi-network card traffic forwarding on the host computer.

### Linux Host

The verification is performed on the `Ubuntu 18.04` system, and other systems may be different. Ubuntu can achieve traffic forwarding through iptables. Before forwarding,
you need to set the USB network card static IP to `172.16.110.5`.

```
nmtui #Enter the graphical interface to set the USB network card static IP

echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -j MASQUERADE

iptables -F #Clear all previous iptables rules
iptables -P INPUT ACCEPT #Allow receiving of data packets
iptables -P FORWARD ACCEPT #Allow sending of data packets

# NAT on the eth0 network port, eth0 is the network card of my PC connected to the external network, configure according to your actual situation
iptables -t nat -A POSTROUTING -s 172.16.110.0/24 -o eth0 -j MASQUERADE
```

### Windows Host

The verification is performed on the ` Windows 10 ` system, and other systems may be different. Windows 10 can directly enable devices to access the Internet through a shared network.

* Install the driver. After connecting the device and turning on the computer, open the device manager and you will see the RNDIS device. Just right-click to update the driver. If the update fails, try using a third-party driver installation tool such as Driver Genius to update the driver.

![](../../../rv1126_img/CAM-C1126S2U/rndis_dev.jpg)

* Set the host's own network card to connect to the external network and share the network.

![](../../../rv1126_img/CAM-C1126S2U/rndis_eth.jpg)

* Set the static IP of the RNDIS network card to ` 172.16.110.5 `.

![](../../../rv1126_img/CAM-C1126S2U/rndis_setting.jpg)