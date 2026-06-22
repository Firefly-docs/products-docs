## CAN
### Introduction
Controller area network (can) is a kind of serial communication network which can effectively support distributed control or real-time control. Can bus is a bus protocol widely used in automobile, which is designed as the communication of microcontroller in automobile environment.
* Check [TI application report for more](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)
### Hardware Connection
Connection between two CAN devices, only need CAN_H to CAN_H, CAN_L to CAN_L.
![](../../../rv1126b_img/CORE-1126BJD4/usage_can.png)

### DTS Configuration
* Common `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b.dtsi`
    ```
    can0: can@21d40000 {
        compatible = "rockchip,rv1126b-canfd";
        reg = <0x21d40000 0x1000>;
        interrupts = <GIC_SPI 204 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&cru CLK_CAN0>, <&cru HCLK_CAN0>;
        clock-names = "baudclk", "apb_pclk";
        resets = <&cru SRST_RESETN_CAN0>, <&cru SRST_HRESETN_CAN0>;
        reset-names = "can", "can-apb";
        dmas = <&dmac 44>;
        dma-names = "rx";
        status = "disabled";
    };

    can1: can@21d50000 {
        compatible = "rockchip,rv1126b-canfd";
        reg = <0x21d50000 0x1000>;
        interrupts = <GIC_SPI 205 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&cru CLK_CAN1>, <&cru HCLK_CAN1>;
        clock-names = "baudclk", "apb_pclk";
        resets = <&cru SRST_RESETN_CAN1>, <&cru SRST_HRESETN_CAN1>;
        reset-names = "can", "can-apb";
        dmas = <&dmac 45>;
        dma-names = "rx";
        status = "disabled";
    };
    ```
Because there is only one CAN device created by the system according to the dts , and the first device created is CAN0.
### Communication
#### CAN communication test
Use the "candump" and "cansend" tools directly to send and receive messages, push tool into /system/bin/ . Tools "candump/cansend" are included with the SDK and download from [Officail link](http://www.t-firefly.com/share/index/index/id/3cacb04c663f9fe97bf494ca55763dcd.html) or [github](https://github.com/linux-can/can-utils).

```
#Close the can0 device at the transceiver
ip link set can0 down
#Set the bit rate to 250Kbps at the transceiver                    
ip link set can0 type can bitrate 250000
#Show can0 details
ip -details link show can0
#Open the can0 device at the transceiver 
ip link set can0 up
#Perform candump on the receiving end, blocking waiting for messages               
candump can0
#Execute cansend at the sending end to send the message                         
cansend can0 123#1122334455667788
```

### More Command
```
1、 ip link set canX down 		//turn off CAN device
2、 ip link set canX up   		//turn on CAN device
3、 ip -details link show canX 		//show CAN device details
4、 candump canX  			//Receive data from CAN bus
5、 ifconfig canX down 			//shutdown CAn device
6、 ip link set canX up type can bitrate 250000 //Set CAN Baudrate
7、 conconfig canX bitrate + (Baudrate)
8、 canconfig canX start 		//start CAN device
9、 canconfig canX ctrlmode loopback on //loopback test
10、canconfig canX restart 		//restart CAN device
11、canconfig canX stop 		//stop CAN device
12、canecho canX 			//check CAN device status查看can设备总线状态；
13、cansend canX --identifier=ID+data 	//send data
14、candump canX --filter=ID:mask 	//Use the filter to receive ID matching data
```

### FAQS
Summarize several problems and solutions encountered during debugging.
#### Check if the CAN_H and CAN_L lines of the bus are loose or connected in reverse.
The receiving end only successfully received the message once, and then no longer received the message.
#### Configuration about clock rate
##### CAN

If the bitrate of CAN is 1M, it is recommended to modify the CAN clock rate to 300M to make the signal more stable. If the bitrate is lower than 1M, the clock rate can be set to 200M.

