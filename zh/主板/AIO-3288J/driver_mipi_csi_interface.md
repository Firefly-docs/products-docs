## 配置原理

设置摄像头相关的引脚和时钟，即可完成配置过程。

从以下摄像头接口原理图可知，需要配置的引脚有：CIF_PWR、MIPI_PWR、MIPI_SDA、MIPI_SCL、MIPI_RST 和 CIF_PDN0。

![](../../../rk3288_img/AIO-3288J/mipi_csi_1.png)

* CIF_PWR 对应 AIO-3288J 上的 GPIO7_B4 引脚：

![](../../../rk3288_img/AIO-3288J/mipi_csi_2.png)

* MIPI_PWR 和 MIPI_RST 对应 AIO-3288J 上的 GPIO3_B2 和 GPIO3_B0 引脚，默认为高电平：

![](../../../rk3288_img/AIO-3288J/mipi_csi_3.png)

* MIPI_SDA 和 MIPI_SCL 由 DVP_PWR 控制，DVP_PWR 对应 GPIO0_B3 引脚：

![](../../../rk3288_img/AIO-3288J/mipi_csi_4.png)
![](../../../rk3288_img/AIO-3288J/mipi_csi_5.png)
![](../../../rk3288_img/AIO-3288J/mipi_csi_6.png)

* CIF_PDN0对应GPIO2_B6引脚：

![](../../../rk3288_img/AIO-3288J/mipi_csi_7.png)

在开发板中，除了 CIF_POWER 和 DVP_POWER 要在 DTS 和驱动中设置，其它引脚都是在 cam_board_rk3288_aio-3288j.xml 中设置。