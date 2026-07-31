# CAN
## Introduction
Controller area network (can) is a kind of serial communication network which can effectively support distributed control or real-time control. Can bus is a bus protocol widely used in automobile, which is designed as the communication of microcontroller in automobile environment.
* check [TI application report for more](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf).

## CAN FD Introduction
Can FD is the abbreviation of can with flexible data rate. It can also be simply regarded as the upgrade of traditional CAN bus. 
Compared with the traditional CAN bus technology, CAN FD has two aspects of upgrade            
* Can FD adopts variable rate, the highest speed can reach 10Mb/s, while the traditional CAN protocol only has 1Mb/s.           
* It can support higher load, and the transmission rate can reach 64 bytes in a single data frame, avoiding the frequent data splitting. 

## Hardware Connection
Connection between two CAN devices, only need CAN_H to CAN_H, CAN_L to CAN_L.

## DTS Configuration
**Only RK3562J has CAN**
* Common DTS `arch/arm64/boot/dts/rockchip/rk3562j.dtsi`
```
/ {
    can0: can@ff600000 {
        compatible = "rockchip,rk3562-can";
        reg = <0x0 0xff600000 0x0 0x1000>;
        interrupts = <GIC_SPI 64 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&cru CLK_CAN0>, <&cru PCLK_CAN0>;
        clock-names = "baudclk", "apb_pclk";
        resets = <&cru SRST_CAN0>, <&cru SRST_P_CAN0>;
        reset-names = "can", "can-apb";
        status = "disabled";
    };

    can1: can@ff610000 {
        compatible = "rockchip,rk3562-can";
        reg = <0x0 0xff610000 0x0 0x1000>;
        interrupts = <GIC_SPI 65 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&cru CLK_CAN1>, <&cru PCLK_CAN1>;
        clock-names = "baudclk", "apb_pclk";
        resets = <&cru SRST_CAN1>, <&cru SRST_P_CAN1>;
        reset-names = "can", "can-apb";
        status = "disabled";
    };
};
```

* Board DTS `arch/arm64/boot/dts/rockchip/rk3562-firefly-aio-3562jq.dtsi`
```
&can0 {
    status = "okay";
    assigned-clocks = <&cru CLK_CAN0>;
    assigned-clock-rates = <200000000>;
    pinctrl-names = "default";
    pinctrl-0 = <&can0m1_pins>;
};
```

## Communication

### CAN FD communication test
Use the "candump" and "cansend" tools directly to send and receive messages. Download tools:
```bash
apt update
apt install -y can-utils
```

Start test:
```bash
#Close the can0 device at the transceiver
ip link set can0 down    

#Set 1Mbps bit rate of arbitration section and 3Mbps dbit rate of data section t the transceiver
ip link set can0 type can bitrate 1000000 dbitrate 3000000 fd on

#Show can0 details
ip -details link show can0

#Open the can0 device at the transceiver
ip link set can0 up

#Perform candump on the receiving end, blocking waiting for messages
candump can0

#canfd format, Execute cansend at the sending end to send the message
cansend can0 123##112345678

#can format, Execute cansend at the sending end to send the message
cansend can0 123#1122334455667788
```

### CAN communication test

Start test
```bash
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

## FAQS
Summarize several problems and solutions encountered during debugging.
### Takes a lone time to receive or can't receive
Check if the CAN_H and CAN_L lines have good and correct connection.

### Configuration about clock rate

If the bitrate of CAN is higher than 1M, it is recommended to modify the CAN clock rate to 300M to make the signal more stable. If the bitrate is lower than 1M, the clock rate can be set to 200M.

Just modify the assigned-clock-rates of can node in dts to change clock rate:
```
assigned-clock-rates = <200000000>;
```

**Note**: 
* under some clock rate frequencies, the bitrate of CAN cannot obtain accurate rate. We can adjust the `assigned clock rates` to solve it 
* Check bitrate

```bash
ip -d link show can1
```

