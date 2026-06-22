## Configuration principles

Set camera-related pin and clock to complete the configuration process.

According to the schematic diagram of camera port below, the pins to be configured include: CIF_PWR, MIPI_PWR, MIPI_SDA, MIPI_SCL, MIPI_RST and CIF_PDN0.

![](../../../rk3288_img/AIO-3288J/mipi_csi_1.png)

* CIF_PWR corresponds to GPIO7_B4 pin on AIO-3288J:

![](../../../rk3288_img/AIO-3288J/mipi_csi_2.png)

* MIPI_PWR and MIPI_RST correspond to GPIO3_B2 and GPIO3_B0 pins on AIO-3288J, the default of which is high level:

![](../../../rk3288_img/AIO-3288J/mipi_csi_3.png)

* MIPI_SDA and MIPI_SCL are controlled by DVP_PWR,DVP_PWR corresponds to GPIO2_B6 pin:

![](../../../rk3288_img/AIO-3288J/mipi_csi_4.png)
![](../../../rk3288_img/AIO-3288J/mipi_csi_5.png)
![](../../../rk3288_img/AIO-3288J/mipi_csi_6.png)

* CIF_PDN0 corresponds to GPIO2_B6 pin:

![](../../../rk3288_img/AIO-3288J/mipi_csi_7.png)

In the development board, except for CIF_POWER and DVP_POWER which shall be set on DTS and driver, the other pins shall be set on cam_board_rk3288_aio-3288j.xml.