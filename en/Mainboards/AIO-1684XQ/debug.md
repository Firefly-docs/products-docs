# Serial debug

USB-to-serial adapter is the abbreviation of USB-to-serial TTL adapter.

## Debugging using DEBUG port

If you want to develop U-Boot or kernel, you need to use the DEBUG port for debugging. The operation is the same as that of RS232 debugging, but you need to pay attention to the adapter and its driver.

### Optional adapter

There are many USB-to-serial adapters on the online store, which are divided into the following types according to the chip:

| Serial port | Maximum baud rate | Recommended or not | Evaluation | Purchase link |
| :--------: | :-------: |:-------: | :-------: | :-------: |
| [CP2104](https://www.firefly.store/products/usb-to-uart-module-cp2104) | 2Mbps | Recommended | Support high baud rate communication, good stability Durable | [Click to buy](https://www.firefly.store/products/usb-to-uart-module-cp2104) |
| CH340 | 2Mbps | Not recommended | In actual use, it is found that the actual baud rate of many CH340s on the market cannot reach 1.5 Mbps | |
| PL2303 | 1.2Mbps | Not recommended | The maximum baud rate cannot reach 1.5Mbps | |

Generally speaking, adapters using CH340 chips have relatively stable performance and are more expensive.

**Note:** The default baud rate for AIO-1684XQ is 115200.

### Hardware connection

USB to Serial adapter with four pins:

* 3.3V power supply (NC), no connection required
* GND, the ground wire of the serial port, connect to the GND pin of the serial port of the development board
* TXD, the output line of the serial port, connected to the TX pin of the serial port of the development board
* RXD, the input line of the serial port, connect to the RX pin of the serial port of the development board

**Note:** If you encounter the problem that TX and RX cannot be input and output when using other serial adapters, you can try to reverse the connection of TX and RX.

AIO-1684XQ DEBUG port:

![](../../../bm1684_img/AIO-1684XQ/debug_connection.png)


### Install driver

Download the driver and install:

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* [PL2303](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)
* [CP210X](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

If you install the CP210X driver from the official website on the Windows system, use tools such as PUTTY or SecureCRT to set the serial port baud rate to 115200. If the setting cannot be set or is invalid, you can download the old version [driver](http://www.t-firefly.com/share/index/index/id/a2e8f25f3d53992bf3e04f45b0e6c8e8.html).

After inserting the adapter, the system will prompt to discover new hardware and initialize it, and then you can find the corresponding COM port in the device manager:

![](../../../bm1684_img/debug_find_com.jpg)