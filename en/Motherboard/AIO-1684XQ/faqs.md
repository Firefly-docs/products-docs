
# FAQs

## Certain USB3 ports do not support USB2.0 devices.

![](../../../bm1684_img/AIO-1684XQ/usb20_no.png)

As shown in the above diagram, among the 4 USB3.0 ports, the USB3.0 port located in the upper left corner is not compatible with USB2.0 devices, while the other 3 USB ports are compatible with USB2.0 devices.

##  5G Compatibility with USB3.0 and SATA
In terms of hardware:

- The SATA 3.0 interface and USB 3.0 share the same PCIe0 lane for expansion.
- The 5G M.2 socket is expanded from PCIe1.  

Both PCIe0 and PCIe1 are expanded from the internal hardware RC_PCIEX. However, these two cannot be used simultaneously; it's not a concept of a hub.

So, when you are using USB3.0 or a SATA hard drive, the 5G functionality cannot be used.

Similarly, when you are using the 5G feature, USB3.0 and SATA hard drives cannot be used simultaneously.## IP configuration of the ethernet interface

![](../../../bm1684_img/AIO-1684XQ/ethernet_interfaces.png)

- Ethernet port 0 (located on the upper layer) is set up with a dynamic IP, which can be obtained automatically via DHCP.  

- Ethernet port 1 (located on the lower layer) is configured with a static IP 192.168.150.1 and subnet mask 255.255.255.0. You can set your PC's IP address to 192.168.150.2/24 for initial access.

If your PC can successfully ping the IP address of the Ethernet port, you can proceed to SSH login. The SSH port is 22, and the username and password are both linaro:

```
ssh linaro@192.168.150.1
```

Once connected to the development board, you can modify the network configuration. The method is similar to Debian/Ubuntu desktop versions. You can configure it in the /etc/netplan/ directory.
## What is the default username and password of the system?

* Username: `linaro`
* Password: `linaro`
* Switch superuser: `sudo -s`

## What should I do if the startup is abnormal and the cycle restarts?

It may be that the power supply current is not enough, please use a power supply with a voltage of 12V and a current of more than 5A.

