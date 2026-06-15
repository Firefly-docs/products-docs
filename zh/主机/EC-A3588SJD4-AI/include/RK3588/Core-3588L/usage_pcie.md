# PCIe 使用

## 简介
AIO-3588SJD4-AI 开发板上有 2 个 PCIe2.0 x 2 (M.2 SATA / PCIe Wifi/BT 模块），如图：

![](../../../rk3588_img/EC-A3588SJD4-AI/usage_pcie_interface.png)

## 软件配置
关于 RK3588 PCIe 的硬件可用资源及软件上 pcie 控制器节点、 PHY 节点对应关系如图：
![](../../../rk3588_img/EC-A3588SJD4-AI/usage_pcie_phy.png)

AIO-3588SJD4-AI 开发板上使用情况如下

* PCIe2.0 x 1 接口两个
    * 资源为`pcie@fe170000` 用做[PCIe WiFi/BT模块](https://wiki.t-firefly.com/zh_CN/Core-3588L/module_wireless.html)

    * 资源为`pcie@fe190000` 默认用作[M.2 SATA](https://wiki.t-firefly.com/zh_CN/Core-3588L/usage_sata.html)，如需切换为切换为PCIe，请查阅软件配置章节[SATA/PCIE](https://wiki.t-firefly.com/zh_CN/Core-3588L/usage_sata.html#ruan-jian-pei-zhi)
