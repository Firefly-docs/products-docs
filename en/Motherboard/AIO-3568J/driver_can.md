## CAN
### Introduction
Controller area network (can) is a kind of serial communication network which can effectively support distributed control or real-time control. Can bus is a bus protocol widely used in automobile, which is designed as the communication of microcontroller in automobile environment.
* check [TI application report for more](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf).

<font color=red>Notices：</font>The CAN FD bit filling function of RK3568 is different from the CAN FD standard protocol, and CAN FD is not recommended.

### Hardware Connection
Connection between two CAN devices, only need CAN_H to CAN_H, CAN_L to CAN_L.

### DTS Configuration
* Common `arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi`

    ```
    &can1 {
            status = "disabled";
            compatible = "rockchip,can-1.0";
            assigned-clocks = <&cru CLK_CAN1>;
            assigned-clock-rates = <150000000>;
            pinctrl-names = "default";
            pinctrl-0 = <&can1m1_pins>;
    };

    &can2 {
            status = "disabled";
            compatible = "rockchip,can-1.0";
            assigned-clocks = <&cru CLK_CAN2>;
            assigned-clock-rates = <150000000>;
            pinctrl-names = "default";
            pinctrl-0 = <&can2m0_pins>;
    };
    ```

* Board `arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dtsi`


    ```
    &can1 {
        status = "okay";
    };

    &can2 {
        status = "okay";
    };
    ```

the nodes on the hardware interface corresponding to the software are:
```
CAN2(H2、L2):can0
CAN1(H1、L1):can1
```

### Communication
#### CAN communication test
Use the "candump" and "cansend" tools directly to send and receive messages, push tool into /system/bin/ . Tools "candump/cansend" download from [Officail link](http://www.t-firefly.com/share/index/index/id/3cacb04c663f9fe97bf494ca55763dcd.html) or [github](https://github.com/linux-can/can-utils).

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
#### Change clock rate
```shell
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi
index 274ebb1..5f2ca7b 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi
@@ -673,7 +673,7 @@
        status = "disabled";
        compatible = "rockchip,can-1.0";
        assigned-clocks = <&cru CLK_CAN1>;
-       assigned-clock-rates = <150000000>;
+       assigned-clock-rates = <100000000>;
        pinctrl-names = "default";
        pinctrl-0 = <&can1m1_pins>;
 };
@@ -682,7 +682,7 @@
        status = "disabled";
        compatible = "rockchip,can-1.0";
        assigned-clocks = <&cru CLK_CAN2>;
-       assigned-clock-rates = <150000000>;
+       assigned-clock-rates = <100000000>;
        pinctrl-names = "default";
        pinctrl-0 = <&can2m0_pins>;
 };
```

**Note**: 
* under some clock rate frequencies, the bitrate of CAN cannot obtain accurate rate. We can adjust the `assigned clock rates` to solve it 
* Check bitrate

    ```
    ip -d link show can1
    ```

