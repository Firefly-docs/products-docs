# Make an SD boot card

Following the previous step of using SD card to upgrade the firmware, this paper mainly introduces how to use a MicroSD card to make an SD boot card for starting and running the system.
You need to follow the same steps to make a bootable card, except that you need to check the **SD Boot** box when you open **SD_Firmware_Tool**, as shown below:

![](../../../rk3399_img/SD_Firmware_Tool_SDBOOT.png)

After selecting the correct removable disk device and firmware update, click **Create**, <font color="#dd0000">Note: Not every device and SDK supports SD boot </font>

Currently, ROC-RK3399-PC-Plus only supports SD boot for **Android7.1** and **Android10.0**.

* Select Android7.1 to make the SD boot card. The firmware can be directly selected from the EMMC upgrade firmware to make the SD boot card，The two are compatible, and when using an SD boot card, the EMMC does not need to be removed. The system will automatically recognize the boot from the card, just make sure that the firmware in the EMMC is the same version of Android7.1.

* When you select the firmware of Android10.0 to make an SD boot card, you need to modify the DTS configuration of the kernel to add the SD card boot function. Please refer to the modifications below：
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
After modification, the fully compiled firmware will support SD boot, but note that this firmware is not compatible with the firmware directly upgraded into the EMMC, cannot be used as an EMMC upgrade firmware, and you will need to remove or erase the EMMC before using the SD boot card.
                                                                                                                                                                               

<font color="#dd0000">Note:</font> Since making an SD boot card rewrites the partitions on the card, you can click **Restore** to get your SD card back to work.