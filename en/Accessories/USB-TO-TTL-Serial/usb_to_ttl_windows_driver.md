# Windows driver installation

## Install Driver

Our usb-to-ttl module is using CP2104, so download driver here:

* [CP210X](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers)

If you bought the module using the CH340 or PL2303 from elsewhere, you can download drivers here:

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* PL2303

If you can’t use PL2303 normally on Win8, use 3.3.5.122 or older version of the old driver, please refer to [This article](https://blog.csdn.net/ropai/article/details/19619951). Please find drivers with version 3.3.5.122 or before.

If you install the CP210X driver from the official website on the Windows system, you can set the serial port baud rate to 1500000 using tools such as PUTTY or SecureCRT. If you cannot set the baud rate or it is invalid, you can download the [old version driver](http://www.t-firefly.com/share/index/index/id/a2e8f25f3d53992bf3e04f45b0e6c8e8.html).

After the adapter is inserted, the system will prompt for the discovery of new hardware and initialization, and then the corresponding COM port can be found in the device manager:

![](../../../modules_img/USB-TO-TTL-Serial/debug_find_com.png)