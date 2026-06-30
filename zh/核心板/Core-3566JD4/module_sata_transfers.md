# [SATA转接板](https://store.t-firefly.com/goods.php?id=154)
AIO-3566JD4使用SATA硬盘时需要用到SATA转接板模块，SATA转接板模块适配Firefly具有FPC SATA接口的所有系列主板，可接2.5”或3.5” SSD/HDD。SATA转接板和AIO-3566JD4采用FPC排线连接，铝箔屏蔽，最大限度减少信号干扰

![](../../../rk356x_img/Core-3566JD4/TF-SATA-TRA_connection.jpg)

## SATA转接板模块实物图

* SATA转接板正面
![](../../../rk356x_img/Core-3566JD4/TF-SATA-TRA_front.jpg)

* SATA转接板背面
![](../../../rk356x_img/Core-3566JD4/TF-SATA-TRA_back.jpg)

## AIO-3566JD4连接SATA硬盘

![](../../../rk356x_img/Core-3566JD4/TF-SATA-TRA_connect_mainboard.jpg)

**注意:** 为防止烧坏的情况发生，板子请先断电再接上SATA硬盘
(公版固件默认关闭SATA功能)

## 启用SATA功能
* 由于AIO-3566JD4上SATA与M.2 PCIe功能复用，使能SATA修改如下：
```
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3566-firefly-aiojd4.dtsi
@@ -173,11 +173,11 @@
     pinctrl-names = "default";
     pinctrl-0 = <&pcie_reset_gpio>;
     /delete-property/ vpcie3v3-supply;
-    status = "okay";
+    status = "disabled";
 };

 &sata2 {
-    status = "disabled";
+    status = "okay";
 };
```

