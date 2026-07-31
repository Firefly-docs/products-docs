# UART

## Introduction

AIO-1126-JD4 supports  UART3, and UART4 interfaces
## DTS Configuration

File path: `kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4v0.dts`

```dts
#ifdef FF_UART3
&uart3 {
        pinctrl-0 = <&uart3m1_xfer_pins>;
        status = "okay";
};
#endif

#ifdef FF_UART4
&uart4 {
        pinctrl-0 = <&uart4m1_xfer_pins>;
        status = "okay";
};
#endif
```

After configuring the serial port, the nodes corresponding to the hardware interface on the software are:
```
UART3:  /dev/ttyS3
UART4:  /dev/ttyS4
```

## Send And Receive Verification

Users can use different host USB to serial port adapters to send and receive data to the serial port of the development board according to different interfaces. For example, the debugging steps of UART3 are as follows:

(1) Development board sends, host receives

```
# Host terminal executes first:
# /dev/ttyUSB0 is the node of host serial port adapter, modify according to actual situation
cat /dev/ttyUSB0

# Development board debug serial port terminal executes:
# /dev/ttyS3 is UART3 node
echo "firefly UART3 test..." > /dev/ttyS3
```

The host terminal can receive the string "firefly UART3 test..."

(2) Host sends, development board receives

```
# Development board debug serial port terminal executes first:
# /dev/ttyS3 is UART3 node
busybox  stty -echo -F /dev/ttyS3       # Turn off echo
cat /dev/ttyS3

# Host terminal execution:
# /dev/ttyUSB0 is the node of the host serial port adapter, modify according to the actual situation
echo "firefly UART3 test..." > /dev/ttyUSB0
```

The development board debugging serial port terminal can receive the string "firefly UART3 test..."
