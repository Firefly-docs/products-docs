## CAN Usage

### Introduction to CAN

CAN (Controller Area Network) bus, also known as the Controller Area Network bus, is a serial communication network that effectively supports distributed control or real-time control. The CAN bus is a widely adopted bus protocol in automobiles, designed for microcontroller communication in automotive environments. For more information, you can refer to[CAN Application Report](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)

### Hardware Connection

CAN on AIO-1688JD4 Development Board [The interface location is shown in the figure](interface_definition.html#zheng-ji-jie-kou-ding-yi)

Since there is only one CAN, it is set by default in the kernel that the first created device is `can0`。

### CAN Communication Testing

Use the candump and cansend tools for message transmission and reception testing. Place the tools in the /system/bin/ directory to execute. The tools are included in the SDK, and can also be found at [Official](http://www.t-firefly.com/share/index/index/id/3cacb04c663f9fe97bf494ca55763dcd.html) Or [github](https://github.com/linux-can/can-utils) Download.

```
#Disable the can0 device on the transmitter and receiver side
ip link set can0 down
#Set bitrate to 250Kbps on both the transmitter and receiver sides.                 
ip link set can0 type can bitrate 250000
#Turn on the can0 device on both the transmitter and receiver side.  	
ip link set can0 up
#Execute candump on the receiving end, block and wait for the message.                        	
candump can0
#Execute cansend on the sender side, send the message.        	
cansend can0 123#1122334455667788  	
```

#### More commands


```
1. ip link set canX down //turn off the can device;
2. ip link set canX up //turn on the can device;
3. ip -details link show canX //show canX device details;
4. candump canX //Receive data from the can bus;
5. ifconfig canX down //Disable the canX device for configuration.
6. ip link set canX up type can bitrate 250000 //set can baud rate
7. conconfig canX bitrate + baud rate;
8. canconfig canX start //start the can device;
9. canconfig canX ctrlmode loopback on //loopback test;
10. canconfig canX restart // Restart the can device;
11. canconfig canX stop // stop the can device;
12. canecho canX //View can device bus status;
13. cansend canX --identifier=ID+data //send data;
14. candump canX --filter=ID:mask //use filter to receive data with matching IDs
```

### FAQS

Summary of several issues encountered during debugging and their solutions:

#### Messages are received long after being sent, or not received at all.

Check the bus CAN\_H and CAN\_L, and ensure that the Dupont wires are not loose or reversed.