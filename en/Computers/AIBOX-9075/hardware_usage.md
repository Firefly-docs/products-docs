# Hardware Function Usage
## Debug Serial Port

The AIBOX-9075 uses an on-board USB serial solution (the serial-to-USB chip is CH342). You can debug the device by connecting the Type-C debug port of the device with a USB cable directly, no external USB to TTL serial module is required.

After the driver is installed on Windows, two serial devices "USB-Enhanced-SERIAL-A CH342" and "USB-Enhanced-SERIAL-B CH342" will appear in the device manager: SERIAL-A is the debug serial port of the main system (Linux), and SERIAL-B is the debug serial port of the subsystem (RTOS). Serial port parameters: baudrate 115200, 8 data bits, 1 stop bit, no parity check.

For the connection method and Windows driver installation, please refer to: [Debug Console](debug.md).

## Display Interface

The AIBOX-9075 provides 1 HDMI 2.0 display output interface. The device runs Ubuntu 24.04 (Wayland + Gnome desktop) by default, and the desktop will be displayed after connecting a monitor.

## Ethernet

The AIBOX-9075 provides 2 x 2.5G RJ45 Ethernet ports, corresponding to the `eth0` and `eth1` devices in the system (subject to the actual system). After the Ethernet port is connected to the network, you can log in to the device through the debug serial port or SSH to check the IP address and test the connectivity:

```
ifconfig eth0
ping -I eth0 -c 10 www.baidu.com
```

## USB Interface

The AIBOX-9075 provides 2 x USB3.0 ports for USB keyboard, mouse, USB flash drive, etc.

## RS485

The AIBOX-9075 provides 2 RS485 ports: RS485_1 corresponds to `/dev/ttyHS1` in the system, and RS485_2 corresponds to `/dev/ttyHS3`. RS485 transmission needs an additional GPIO to control the direction (RS485_1 uses GPIO No.695, RS485_2 uses GPIO No.693). For the transmission test, please refer to [Serial Tutorial](usage_serial.md).

## CAN-FD

The AIBOX-9075 provides 2 CAN-FD interfaces (optocoupler isolated) for communication with CAN bus devices. Detailed usage is to be supplemented.

## GMSL2

The AIBOX-9075 provides 2 x GMSL2 4Pin Mini FAKRA interfaces for connecting GMSL2 cameras and other devices. Detailed usage is to be supplemented.

## IO

The AIBOX-9075 provides 6 IO input channels (IO in) and 6 IO output channels (IO out). Detailed usage is to be supplemented.

## SIM Card

The SIM card slot of the AIBOX-9075 is used together with a wireless module to provide mobile network connectivity. **The wireless module is optional**, and the SIM card function is available only after the module has been installed inside the chassis. The SIM card insertion direction is shown in the figure below. Please power off the device before inserting or removing the SIM card.

<center>

<img alt="" src="../../../qcom_img/AIBOX-9075/sim_insert_direction.png" width="400">
</center>

## RTC

The AIBOX-9075 supports the RTC (Real Time Clock). For reading, writing and synchronizing the time, please refer to [RTC Tutorial](usage_rtc.md).

## Watchdog

The AIBOX-9075 supports the hardware watchdog. For enabling and feeding the watchdog, please refer to [Watchdog Tutorial](usage_watchdog.md).

## Video

The default video framework of the AIBOX-9075 is Gstreamer. For the codec capabilities and playback/encoding commands, please refer to [Video Tutorial](usage_video.md).

## NPU (AI)

The IQ-9075 delivers a peak computing power of 200 TOPS and supports private deployment of edge-side large models. For AI development, please refer to [AI Tutorial](usage_npu.md).
