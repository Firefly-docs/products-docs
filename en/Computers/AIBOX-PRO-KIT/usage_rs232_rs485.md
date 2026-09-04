# RS485

AIBOX-PRO-KIT has one RS485 interface. If the CPU is RK3588, the device name is `/dev/ttyS6`; if the CPU is RK3576, the device name is `/dev/ttyS3`. It supports half-duplex communication, and the default baud rate is `9600`. This interface uses a Phoenix terminal block, so a compatible terminal connector is required. Once connected, it can be debugged using standard serial port methods.

