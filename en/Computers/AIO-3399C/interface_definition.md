# Interface definition

## AIO-3399C interface definition

AIO-3399C provides rich interfaces, including :

* Power interface,
* Type - C (OTG),
* 1 x USB3.0, 4 x USB2.0 (interface 2, seat 2),
* HDMI,
* Ethernet,
* Double an LVDS interface screen, eDP screen interface,
* TP touch interface,
* Screen voltage jump line interface,
* Backlight interface,
* WIFI antenna,
* Bluetooth antenna,
* Power button,
* MIC interface,
* Audio input/output (linein, lineout), 3.5 mm headphone jack,
* Power RTC interfaces,
* 12 v power supply interface,
* IR interface,
* TF card slot, SIM card slot,
* Interface extension buttons,
* I2C,I2S，
* Speaker interface,
* USB2.0 HOST,
* Recovery button,
* EDP screen interface,
* debugging serial port,
* Industrial serial port (RS485,RS232,2TTL),
* RTC power interface,
* MIPI screen interface (dual LVDS multiplex),
* mini-pcie.

The details are as follows:

![](../../../rk3399_img/AIO-3399C/interface_details.en.jpg)

*In addition, customers can also customize the boards of relevant functional interfaces as required, as shown in the figure below:*

![](../../../rk3399_img/AIO-3399C/interface_customized.en.jpg)

Beside interface above，AIO-3399C(AI) support NPU speeding for it drive SPR2801S .Howerver, AIO-3399C(AI) do not support all Type-C function (ADB, OTG, DP display)，but it can be used to download firmware with Type-C.

***Note：The hardware version of AIO-3399C(AI) without SPR2801S dosen't support function of npu speeding up. To use the function of NPU please choose and buy the hardware version of AIO-3399C(AI) with the chip of SPR2801S.***

![](../../../rk3399_img/AIO-3399C/interface_with_npu.en.jpg)