# PCIe 使用

## 简介
ROC-RK3588-RT 开发板上有 3 个 PCIe2.0 x 3 接口 和 1 个 PCIe3.0 x 4 接口，其中 2 个 PCIe2.0 x 2 （M.2 SATA / PCIe Wifi/BT 模块），PCIe3.0 x 4 引出接口给扩展板使用，

板载默认支持[M.2 SATA](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-RT/usage_sata.html) 和 [PCIe WiFi/BT模块](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-RT/module_wireless.html)。

如图：

![](../../../rk3588_img/ROC-RK3588-RT/usage_pcie_interface.png)


## 软件配置
关于 RK3588 PCIe 的硬件可用资源及软件上 pcie 控制器节点、 PHY 节点对应关系如图：
![](../../../rk3588_img/ROC-RK3588-RT/usage_pcie_phy.png)

ROC-RK3588-RT 开发板上的 PCIe3.0 x 4 接口使用了 RK3588 的 `PCIe Gen3 x 4 lane` 和 `PCIe Gen3 x 2 lane`  这组资源。
### DTS 配置
一般根据原理图在 DTS 中配置供电引脚、复位引脚，选择正确的 pcie 控制器节点和 PHY 节点使能就可以。

示例： 

以下是一段 [4 x 2.5G 网口扩展板]() 的DTS配置 `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-rt-ext.dtsi` ：
```
/* pcie3.0 x 4 Lane */
&pcie30phy {
    rockchip,pcie30-phymode = <PHY_MODE_PCIE_NABIBI>;
    status = "okay";
};

&pcie3x2 {
        num-lanes = <1>;
        reset-gpios = <&gpio3 RK_PD4 GPIO_ACTIVE_HIGH>;
        vpcie3v3-supply = <&vcc3v3_pcie30>;
    rockchip,skip-scan-in-resume;
    rockchip,perst-inactive-ms = <500>;
    status = "okay";
};
&pcie3x4 {
        num-lanes = <1>;
        reset-gpios = <&gpio4 RK_PB6 GPIO_ACTIVE_HIGH>;
        vpcie3v3-supply = <&vcc3v3_pcie30>;
    rockchip,skip-scan-in-resume;
    rockchip,perst-inactive-ms = <700>;
    status = "okay";
};

&vcc3v3_pcie30{
        gpios = <&gpio1 RK_PB2 GPIO_ACTIVE_HIGH>;
        regulator-always-on;
        startup-delay-us = <5000>;
        status = "okay";
};
```
`pcie30phy`：PHY 节点

`pcie3x2`：pcie3x2 控制器节点

`pcie3x4`：pcie3x4 控制器节点

`reset-gpios`：复位引脚属性

`vcc3v3_pcie30`：供电引脚节点

#### [检测 2.5G 网口是否生效](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-RT/usage_ethernet.html#cha-kan-ip-di-zhi)
