# Debug Console

Debug serial port is very useful during debugging and troubleshooting, especially when the GUI is unavailable.

## Connection

Use USB cable to connect the device `Console` port with PC.

![](../../../qcom_img/AIBOX-9075/debug_console.jpg)

## Install Driver

The serial-to-USB chip is CH342.

Linux PC don't need to install driver.

Windows PC needs driver, please download `CH343SER.EXE` from [Download Link](https://en.t-firefly.com/doc/download/413.html#other_815)

Double click the exe file to install.

## Usage

After driver installation, you will find two serial port "USB-Enhanced-SERIAL-A CH342" and "USB-Enhanced-SERIAL-A CH342" in Windows device manager.

The SERIAL-A is the debug port of main Linux system, and the SERIAL-B is the debug port of RTOS subsystem.

In Linux it will be /dev/ttyUSBX or /dev/ttyACMX, the number X may be different, you can unplug and plug again to find the corresponding device.

Use the serial tool like MobaXterm or Minicom to open the serial device, baudrate is 115200, 8 data bit, 1 stop bit, no parity check.