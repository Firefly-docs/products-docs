# Windows 上制作 SD 启动卡

承接上一步的使用SD卡来更新固件，本文主要介绍如何实现使用MicroSD卡，制作用于启动并运行系统的SD启动卡。
制作启动卡的准备工作和操作步骤一致，唯一一点不同的是，在打开**SD_Firmware_Tool** 的时候，需要勾选的是**SD Boot**框，如下图所示：

![](../../../rk3399_img/SD_Firmware_Tool_SDBOOT.png)

选择正确的可移动磁盘设备和升级固件后，点击开始创建即可, <font color="#dd0000">注意：不是每一款设备和SDK都支持SD启动卡的功能</font>

目前 ROC-RK3399-PC-PLUS仅支持 **Android7.1**,**Android10.0** 做SD卡启动。

* 选择 Android7.1 制作SD启动卡，固件可以直接选择EMMC升级固件来做，二者是兼容的，且在使用SD启动卡的时候，不需要摘掉EMMC，系统会自动识别从卡启动，只需要确保此时EMMC里面的固件是同版本的Android7.1。
* 选择 Android10.0 的固件制作SD启动卡的时候，需要修改内核的DTS配置来添加支持SD卡启动功能，参照修改如下：
```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dts
index 6b7b451cee1..295086a0025 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus.dts
@@ -85,3 +85,7 @@
     assigned-clock-parents = <&cru PLL_CPLL>; 
 };
 
+&sdmmc{
+supports-emmc;
+};
+

```
修改之后，全编译生成的固件即可支持SD启动功能，但注意，该固件和直接烧录进EMMC的固件是不兼容的，不能用作EMMC升级固件，且在使用该SD启动卡的时候，需要先摘掉EMMC或者擦除EMMC。


<font color="#dd0000">注意</font>:因为做SD启动卡会重写卡上的分区，所以如果想恢复SD卡的正常使用，可以点击SD_Firmware_Tool的**Restore**功能即可。