# Debug Console

Debug serial port is very useful during debugging and troubleshooting, especially when the GUI is unavailable.

## Connection

* 3pin TTL socket

<center>
<img alt="" src="../../../qcom_img/Firefly-Q6490A/debug_console.jpg" width="700">
</center>

It needs additional usb-to-ttl module, please refer to [Serial Module](../../Accessories/USB-TO-TTL-Serial/usb_to_ttl_product.md)

## Install Driver

Linux PC don't need to install driver.

Windows PC driver installation is also described in [Serial Module](../../Accessories/USB-TO-TTL-Serial/usb_to_ttl_product.md)

## Usage

After driver installation, you will find "Silicon Labs CP210x USB to UART Bridge" in Windows device manager.

In Linux it will be /dev/ttyUSBX or /dev/ttyACMX, the number X may be different, you can unplug and plug again to find the corresponding device.

Use the serial tool like MobaXterm or Minicom to open the serial device, baudrate is 115200, 8 data bit, 1 stop bit, no parity check.