


## Switch recording input source
Firefly-RK3399 The default recording input source is an onboard microphone `Builtin Mic`,You can manually switch to phone recording `Wired Headset`:

1. Find `sound` in `settgins APK` and click it;
2. Click `advanced` and the `audio input` option will appear;
3. Select `wired header`

![](../../../rk3399_img/faqs_android_audio_input.png)

## How to forcefully enter MaskRom mode

If the board cannot enter the Loader mode, you can try to enter the MaskRom mode forcibly. Please refer to ["MaskRom mode"] (04-maskrom_mode.md) for the operation method.

## PCIE
* What is the difference between the two PCIEs on the development board?
   * The front is PCIe 1.0, M.2, and the back is USB to Mini PCIe. SSD, SATA connect to M.2, LTE/3G connect to Mini PCIe.

* M2 interface type:
   * B-key

* How to connect SSD:
  * The SSDs on the market are basically M-key. If you want to connect to the SSD, you need to go to [Mall](https://www.firefly.store/products) to purchase B-key to M-key Adapter board

* Does SSD support NVME:
    * Support, tested Intel (Intel) 600P series, Samsung EVO series

* How to connect SATA hard disk:
    * If you want to connect to SATA, you need to go to [Mall](https://www.firefly.store/products/pcie-m-2-to-sata3-0-adapter-board) to purchase a PCIE to SATA adapter board


## TYPE-C

* How to connect the monitor?
  * Adapter is required. We have tested adapters for DP and HDMI:
  * Type-C to DP: [Purchase reference link](https://detail.tmall.com/item.htm?id=531442057703&areaId=442000&user_id=1127317597&cat_id=2&is%20_b=1&rn=4eff2fc1aac30e8c67ff74ef5fe76b56)
  * Type-C to HDMI: [Purchase reference link](https://item.taobao.com/item.htm?spm=a1z0d.6639537.1997196601.291.150IpW&id=540645282055&qq-pf-to=pcqq.temporaryc2c)
  * Similar to VGA and DVI are also available, we have not bought a similar adapter, customers can verify by themselves

* Maximum display resolution
  * The default maximum is 2K, if you need 4K, you need to modify the software

* Show is it possible under Linux
  * Currently only supported by Android, the original Linux driver is still under debugging

* Can it be displayed simultaneously with the onboard HDMI
  * Can be displayed at the same time

* Does it support OTG (UFP/DFP)?
  * Support, need to purchase 3.0 adapter cable, purchase [reference link](https://detail.tmall.com/item.htm?spm=a220o.1000855.w5003-14913680624.1.W2eSKK&id=528676463455&scene=taobao_shop&skuId=3173247769269)

## BT (Bluetooth)

* Does the development board support Bluetooth headsets?
  * stand by

## LTE/4G

* Currently supported models:
  * EC20

## Camera

* Does it support dual cameras?
  * Support, currently we have debugged and verified OV13850

* Does it support USB camera?
  * stand by

* How many USB cameras can be supported at most
  * In theory, the USB ports on the board support the camera, but considering the limitation of only one YUV and encoding on a hub, there is no problem with the two

* Supports two MIPI cameras, can one DVP camera be used at the same time?
  * Not supported, due to software architecture issues, currently only supports up to two simultaneous use

## Fan

* Does it support speed control?
  * The current hardware does not support, only supports detection of running status

## RTC

* The time is not synchronized after the development board is powered on
  * Check if the RTC battery is correctly connected.

* Does it support scheduled boot?
  * Support, please see [wiki](http://wiki.t-firefly.com/zh_CN/Firefly-RK3399/driver_rtc.html) for details

