# PCIe

## Introduction
There are 2 PCIe 2.0 x 2 (M.2 SATA/PCIe WiFi/BT modules) on the AIO-3588SG development board, as shown in the figure:

![](../../../rk3588_img/Core-3588SG/usage_pcie_interface.png)

## Software configuration
The available hardware resources of RK3588 PCIe and the corresponding relationship between the `pcie` controller node and PHY node on the software are shown in the figure:
![](../../../rk3588_img/Core-3588SG/usage_pcie_phy_en.jpg)

 The usage on the AIO-3588SG development board is as follows

* Two PCIe 2.0 x 1 interfaces
    * Resources are` pcie@fe170000 `Used as a [PCIE WiFi/BT module](https://wiki.t-firefly.com/en/Core-3588L/module_wireless.html)

    * Resources are` pcie@fe190000 `Default used as [M.2 SATA](https://wiki.t-firefly.com/en/Core-3588L/usage_sata.html),To switch to PCIe, please refer to the software configuration chapter [SATA/PCIE](https://wiki.t-firefly.com/en/Core-3588L/usage_sata.html#software-configurationi)