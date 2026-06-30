According to the schematic diagram below, you need to provide: `CIF_PWR`, `DVP_PWR` and `MIPI_RST`.

* MIPI interface

![](../../../rk356x_img/IPC-M10R800-A3568J/camera_mipi_interface.jpg)

* `DVP_PWR` corresponds to `GPIO1_C1` of RK3399.
* `CIF_PWR` corresponds to `GPIO1_A1` of RK3399.
* `MIPI_RST` corresponds to `GPIO0_B0` of RK3399.

In the development board, all the pins are configured in `cam_board.xml`.