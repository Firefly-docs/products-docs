# Hardware Interfaces

## Board Interface Specification

**EXT-iCore-3576Q38 V1.0** interfaces include:

* 1 x HDMI2.1 (Up to 4K@120Hz)
* 1 x USB Type-C (For download/debug)
* 2 x USB3.0
* 2 x MIPI-DPHY-CSI (CSI0 muxed with CSI1, can not be used at same time)
* 1 x TF Card
* 1 x Mini PCIe (4G module)
* 1 x PCIe M.2 (5G module)
* 1 x SIM Card Slot
* 1 x WIFI (External module)
* 1 x Bluetooth (External module)
* 1 x FAN
* 1 x Headphone （Supports microphone recording, US standard）
* 1 x CAN  (Phoenix connector)
* 1 x RS232 (Phoenix connector)
* 1 x RS485 (Phoenix connector)
* 1 x Line-Out (Through pin header)
* 1 x Line-In (Through pin header)
* 1 x USB2.0 (Through pin header)
* 1 x SPI  (Through pin header)
* 2 x I2C  (Through pin header)

* 1 x MIPI-D/CPHY-CSI
* 2 x RJ45 (Support 1Gbps ethernet)

具体如下图：

![](../../../rk3576_img/iCore-3576Q38/interface_front_en.png)

![](../../../rk3576_img/iCore-3576Q38/interface_back_en.png)

![](../../../rk3576_img/iCore-3576Q38/interface_io_en.png)

## Special Notice
Mini PCIe (4G Module) and PCIe M.2 (5G Module) used same USB bus, so they can not be used at same time.

Default 4G module, you need to change some resistors to use 5G module.