## CAN

## CAN Usage

### CAN Introduction

CAN (Controller Area Network) is a serial communication network that supports distributed and real-time control. For more information, refer to the [CAN Application Report](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf).

### Hardware Connection

The CAN [interface location of the AIBOX-PRO development board is shown here](interface_definition.md).

Since there is only one CAN interface, the first device created in the kernel is `can0` by default.

### CAN Communication Test

Use `candump` and `cansend` to send and receive messages. These tools are included in the SDK and can also be downloaded from [GitHub](https://github.com/linux-can/can-utils).

```
# Bring down can0 on both ends
ip link set can0 down
# Set the bitrate to 250Kbps on both ends
ip link set can0 type can bitrate 250000
# Bring up can0 on both ends
ip link set can0 up
# Run candump on the receiving end
candump can0
# Run cansend on the sending end
cansend can0 123#1122334455667788
```

### FAQS

#### Messages are delayed or not received

Check whether the CAN_H and CAN_L bus wires are loose or connected in reverse.


