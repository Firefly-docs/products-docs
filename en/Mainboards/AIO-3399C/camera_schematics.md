According to the schematic diagram below, you need to provide: `AF_VDD28`, `DOVDD18`, `AVDD28`, `DVDD12`, `PWDN1`, `RST` and `MCLK`.

* MIPI interface

![](../../../rk3399_img/AIO-3399C/camera_mipi_interface.jpg)

* DVP interface

![](../../../rk3399_img/AIO-3399C/camera_dvp_interface.jpg)

* `AF_VDD28` is provided by hardware connection. No configuration is needed.
* `DOVDD18`,`AVDD28`: Controlled by `DVP_PWR`, `DVP_PWR` corresponds to GPIO1_C7 of RK3399:

![](../../../rk3399_img/AIO-3399C/camera_dvp_pwr.jpg)

* `DVDD12` is controlled by `CIF_PWER`, `CIF_PWER`corresponds to GPIO1_C6 of RK3399:

![](../../../rk3399_img/AIO-3399C/camera_cif_pwr.jpg)

* `MIPI CIF`: `PWDN0`(share), `PWDN1`, `RST` corresponds to GPIO2_B4, GPIO2_D4, GPIO0_B0:

![](../../../rk3399_img/AIO-3399C/camera_mipi_cif.jpg)

All the pins are configured in `cam_board.xml`, with the exception of `DVDD12 (CIF_POWER)`, which is configured in DTS and driver.