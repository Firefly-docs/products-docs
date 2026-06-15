# 接口定义

## 整机接口定义

**ROC-RK3576-PC V1.0** 使用的接口，主要包括：

* 1 x HDMI2.1（最大支持 4K@120Hz 输出）
* 1 x USB Type-C (下载/Debug)
* 2 x USB3.0
* 2 x MIPI-DPHY-CSI (CSI0 和 CSI1 不能同时使用)
* 1 x TF Card
* 1 x Mini PCIe (4G 模块)
* 1 x PCIe M.2 (5G 模块)
* 1 x SIM 卡槽
* 1 x WIFI (外置模块)
* 1 x Bluetooth (外置模块)
* 1 x FAN
* 1 x Headphone （支持MIC录音，美标）
* 1 x CAN  (凤凰端子座)
* 1 x RS232 (凤凰端子座)
* 1 x RS485 (凤凰端子座)
* 1 x Line-Out (由排针引出)
* 1 x Line-In (由排针引出)
* 1 x USB2.0 (由排针引出)
* 1 x SPI  (由排针引出)
* 2 x I2C  (由排针引出)

* 1 x MIPI-D/CPHY-CSI
* 2 x RJ45 (支持 1Gbps 以太网)


具体如下图：

![](../../../rk3576_img/EC-R3576PC-FD/interface_front_zh.png)

![](../../../rk3576_img/EC-R3576PC-FD/interface_back_zh.png)

![](../../../rk3576_img/EC-R3576PC-FD/interface_io_zh.png)

## 特殊接口说明
Mini PCIe (4G 模块) 和 PCIe M.2 (5G 模块) 共用了一路 USB，所以不能同时使用。

默认是 4G 模块，如需使用 5G 需要调整底板电阻位置。