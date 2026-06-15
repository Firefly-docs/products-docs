从以下摄像头接口原理图可知，需要配置的引脚有：`AF_VDD28`、`DOVDD18`、`AVDD28`、`DVDD12`、`PWDN1`、`RST` 和 `MCLK`。

* MIPI 接口

![](../../../rk3399_img/AIO-3399C/camera_mipi_interface.jpg)

* DVP 接口

![](../../../rk3399_img/AIO-3399C/camera_dvp_interface.jpg)

* `AF_VDD28` 可不做配置。
* `DOVDD18`、`AVDD28` 由 `DVP_PWR` 控制，`DVP_PWR` 对应 RK3399 的 GPIO1_C7：

![](../../../rk3399_img/AIO-3399C/camera_dvp_pwr.jpg)

 * `DVDD12` 由 `CIF_POWER` 引脚控制，`CIF_POWER` 对应 RK3399 上的 `GPIO1_C6` 引脚：

![](../../../rk3399_img/AIO-3399C/camera_cif_pwr.jpg)

 * `MIPI CIF`：PWDN0(共用)、PWDN1、RST 对应 GPIO2_B4、GPIO2_D4、GPIO0_B0 引脚：

![](../../../rk3399_img/AIO-3399C/camera_mipi_cif.jpg)

在开发板中，除了 `DVDD12 (CIF_POWER)` 要在 DTS 中设置以外，其它引脚都是在 `cam_board.xml` 中设置。