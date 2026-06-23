
# Serial port module

## [USB to TTL serial port module](https://www.firefly.store/products/usb-to-uart-module-cp2104)

### Product parameters

* Brand: Firefly
* Size: 25mm*15.5mm

### Technical information

Driver download: [http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)

### Physical map

![](../../../aibox_img/module_serial_list.png)

### Connection method

USB to Serial adapter with four pins:

* 3.3V power supply (NC), no connection required
* GND, the ground wire of the serial port, connect to the GND pin of the serial port of the development board
* TXD, the output line of the serial port, connected to the TX pin of the serial port of the development board
* RXD, the input line of the serial port, connect to the RX pin of the serial port of the development board

**Note:** If you encounter the problem that TX and RX cannot be input and output when using other serial adapters, you can try to reverse the connection of TX and RX.

AIBOX-K3 DEBUG port:

![](../../../aibox_img/AIBOX-K3/debug_connection.png)
