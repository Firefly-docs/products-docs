# Quick use

## IP address acquisition

There are three ways to obtain an IP address. It is recommended to use the first method for device access:

* **The first method: The machine is set with the main network card IP address and subnetwork card IP address. The main network card `eth0` uses dhcp to dynamically obtain an IP address to set the IP address. The subnet card `eth0:1` is set with a static IP address by default to facilitate user access. Static IP address of the subnet card: `192.168.1.10`. PCs in the same network segment can directly access the device.**

For example, the IP address of the main network card and the IP address of the subnetwork card are configured as follows:
```bash
# ifconfig 
eth0      Link encap:Ethernet  HWaddr 7A:97:2D:EE:4F:8F
          inet addr:168.168.18.16  Bcast:168.168.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:69277 errors:0 dropped:2135 overruns:0 frame:0
          TX packets:11945 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5251929 (5.0 MiB)  TX bytes:10589068 (10.0 MiB)
          Interrupt:46

eth0:1    Link encap:Ethernet  HWaddr 7A:97:2D:EE:4F:8F
          inet addr:192.168.1.10  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          Interrupt:46
```
* The second method: Access the serial port and enter the terminal to use the ifconfig eth0 command to obtain the IP address
* The third method: Disassemble the shell and connect the USB cable. Use adb shell to enter the board terminal and use the ifconfig eth0 command to obtain the IP address.


## LAN Preview (RTSP)

* The default factory firmware is IPC firmware. The rkipc application runs by default on boot. On a PC within the local area network, the camera's RTSP video stream can be previewed directly.

## Video preview

The device supports previewing in the same LAN. After the device is connected to the Internet, use the PC-side RTSP software (such as VLC) to open network streaming. Execute the button in the upper corner:
Media (M) --> Open network streaming (N) --> Network (N) --> Please enter the network URL

Enter the following address:
```
rtsp://(your device’s IP address)/live/0
```

![](../../../rv1106_img/CT36L/rtsp_preview.png)

## Web page preview

After obtaining the IP address, enter the IP address of the device in the PC browser to enter the web login page. The account number and password are both: admin.

**Note: If the web page preview camera screen remains black, please use the VLC software described in the [Video preview] chapter to preview the camera.**

![](../../../rv1106_img/CT36L/login_in.png)

The preview effect is as follows:
![](../../../rv1106_img/CT36L/web_preview.png)