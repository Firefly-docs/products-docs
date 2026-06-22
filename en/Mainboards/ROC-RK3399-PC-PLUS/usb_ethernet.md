## USB Ethernet

USB Ethernet, the main realization is to use the OTG interface of the device as a peripheral mode and simulate it as a network interface, and then the host connects to the device via USB and accesses the Internet through the device. The following is the specific operation based on Firefly-RK3399 device.

Operating environment:

* PC with Ubuntu system
* Firefly-RK3399 device


### Kernel Settings

In the kernel directory, open the kernel configuration options menu:

```
make firefly_linux_defconfig
make menuconfig
```

After entering the kernel configuration menu, select in turn: `Device Drivers` -> `USB Support` -> `USB Gadget Support`.

Set the `USB Gadget Driver` to compile into a module, and then you can find the option of `Ethernet Gadget (with CDC Ethernet support)` below, and choose to compile into a module as well. At the same time, select `RNDIS support`.

```
<M> USB Gadget Drivers
<M> USB functions configurable through configfs
<M> Ethernet Gadget (with CDC Ethernet support)
[*] RNDIS support (NEW)
```

Then compile the kernel in the `kernel` directory:

```
make rk3399-firefly.img -j12
```

After the compilation is completed, burn the kernel to the device. For the burn process, please refer to the **Upgrade Firmware** page of the corresponding board wiki. Then copy the following modules generated in the kernel directory to the device:

* drivers/usb/gadget/function/u_ether.ko
* drivers/usb/gadget/function/usb_f_ecm_subset.ko
* drivers/usb/gadget/function/usb_f_ecm.ko
* drivers/usb/gadget/function/usb_f_rndis.ko
* drivers/usb/gadget/function/usb_f_eem.ko
* drivers/usb/gadget/legacy/g_ether.ko
* drivers/usb/gadget/libcomposite.ko

Then on the device, load the above modules in sequence:

```
insmod libcomposite.ko
insmod u_ether.ko
insmod usb_f_ecm_subset.ko
insmod usb_f_rndis.ko
insmod usb_f_ecm.ko
insmod usb_f_eem.ko
insmod g_ether.ko
```

**Note:** You must load `libcomposite.ko` and `u_ether.ko` first, and then the following modules can be loaded.

### IP address settings

Connect the PC and the OTG interface of the device with a data cable, and execute the `lsusb` command in the PC to view the USB Ethernet device, which means the connection is successful.

```bash
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

`ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget` in the output information is the USB network card device.

The device is plugged into the network cable so that the device can connect to the external network.

* IP settings in the device:

Enter and execute the `ifconfig -a` command, you can see the following information:

```bash
root@firefly:~# ifconfig -a

# eth0 is the wired network card
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
        inet 168.168.100.48 netmask 255.255.0.0 broadcast 168.168.255.255
        inet6 fe80::1351:ae2f:442e:e436 prefixlen 64 scopeid 0x20<link>
        ether 8a:4f:c3:77:94:ac txqueuelen 1000 (Ethernet)
        RX packets 9759 bytes 897943 (897.9 KB)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 236 bytes 35172 (35.1 KB)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
        device interrupt 42 base 0x8000

        ...

# usb0 is a virtual usb network card
usb0: flags=4098<BROADCAST,MULTICAST> mtu 1500
        ether 4a:81:b1:34:d2:ad txqueuelen 1000 (Ethernet)
        RX packets 0 bytes 0 (0.0 B)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 0 bytes 0 (0.0 B)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

Then customize an appropriate IP for the usb0 network card:

**Note that the IP of usb0 and the IP of wired network card eth0 are not in the same network segment**! !

```
ifconfig usb0 192.168.1.101
```

* IP setting in PC:

```bash
# First check the USB virtual network card
firefly@Desktop:~$ ifconfig
enp0s20u2i1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
        inet 192.168.2.90 netmask 255.255.255.0 broadcast 192.168.2.255
        inet6 fe80::871c:b87e:1327:7fd4 prefixlen 64 scopeid 0x20<link>
        ether 46:fe:6e:97:ee:a6 txqueuelen 1000 (Ethernet)
        RX packets 0 bytes 0 (0.0 B)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 1 bytes 54 (54.0 B)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
...


# Set the USB network card's IP address and the device's usb0 IP address in the same network segment
firefly@Desktop:~$ sudo ifconfig enp0s20u2i1 192.168.1.100

#Set the default gateway: it should be set to the ip address of the device usb0, because the traffic will be forwarded through usb0 later
firefly@Desktop:~$ sudo route add default gw 192.168.1.101
```

After setting the IP of the device and the PC, they can ping each other, and the PC can also use the `ssh` command to log in to the device.

### Network sharing to realize PC Internet access

On the device: First turn on the IPv4 forwarding function:

```bash
echo 1> /proc/sys/net/ipv4/ip_forward
```

If you want to automatically turn on the forwarding function every time you restart the device, please directly modify the value of `net.ipv4.ip_forward` in the `/etc/sysctl.conf` file to 1. After modifying the file parameters, execute the `sysctl -p` command to reload the `/etc/sysctl.conf` file to make the IPv4 forwarding function effective.

Add rules for traffic forwarding:

```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 168.168.100.48
```

Now the host PC can access the network. If the PC can ping the usb0 and eth0 of the device, but cannot access the Internet, you need to modify the DNS of the PC and add the following in `/etc/resolv.conf`:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Pay attention to the following points during the configuration process:

* Correspond to each IP address on your device in the above steps, and note that the USB virtual network card IP and wired network IP on the device are not in the same network segment;
* The gateway of the PC virtual USB network card should be set to the IP address of the device virtual USB network card;
* The virtual network card IP address, IP forwarding function, traffic forwarding rules and other settings on the device will be restored after the device is restarted. The user can find out the information on how to automatically set these configurations after booting.