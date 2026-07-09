# UART

## Introduction

AIO-1126-JD4 supports UART1, UART3, and UART4 interfaces
## DTS Configuration
File path: `kernel/arch/arm/boot/dts/rv1126-firefly-rk809.dtsi`
```
&uart1 {
        status = "okay";
};

&uart3 {
        pinctrl-0 = <&uart3m2_xfer &uart3m2_ctsn &uart3m2_rtsn>;
        status = "okay";
};

&uart4 {
        pinctrl-0 = <&uart4m1_xfer>;
        status = "okay";
};
```

After configuring the serial port, the nodes corresponding to the hardware interface on the software are:
```
UART1:  /dev/ttyS1
UART3:  /dev/ttyS3
UART4:  /dev/ttyS4
```

## Send And Receive Verification

Users can use different host USB to serial port adapters to send and receive data to the serial port of the development board according to different interfaces. For example, the debugging steps of UART1 are as follows:

(1) Development board sends, host receives

```
# Host terminal executes first:
# /dev/ttyUSB0 is the node of host serial port adapter, modify according to actual situation
cat /dev/ttyUSB0

# Development board debug serial port terminal executes:
# /dev/ttyS1 is UART1 node
echo "firefly UART1 test..." > /dev/ttyS1
```

The host terminal can receive the string "firefly UART1 test..."

(2) Host sends, development board receives

```
# Development board debug serial port terminal executes first:
# /dev/ttyS1 is UART1 node
busybox  stty -echo -F /dev/ttyS1       # Turn off echo
cat /dev/ttyS1

# Host terminal execution:
# /dev/ttyUSB0 is the node of the host serial port adapter, modify according to the actual situation
echo "firefly UART1 test..." > /dev/ttyUSB0
```

The development board debugging serial port terminal can receive the string "firefly UART1 test..."