# Switching Upgrade Storage
## Introduction

This article describes what you should pay attention to when burning firmware when the host is attached to both the eMMC(default) and the NOR Flash storage media:

As shown in the figure below, the device has a Nor Flash storage media.

![](../../../rk356x_img/IPC-M10R800-A3568J/nor_flash-position.jpg)

## Loader Mode 

If you enter the loader download mode by executing 'reboot loader' in software or by using the hardware Recovery key, you don't need to look at the next content, just skip to the operation of  [《upgrade_firmware》](03-upgrade_firmware.md) .


## Maskrom Mode

If you enter Maskrom mode through ["How to enter MaskRom mode"](04-maskrom_mode.md), then if you want to burn the entire firmware, please refer to Figure 1 below.And pay attention to the following steps:

![](../../../rk356x_img/maskrom_with_flash.png)

1.First, upgrade the MiniLoaderAll.bin into the machine,The Loader file can be found in the firmware generation path in the SDK directory, such as'android11\rockdev\Image-rk3566_roc_pc\MiniLoaderAll.bin',You need to find the appropriate device model, if you have not compiled the SDK, you can download a [MiniLoaderAll.bin] for the corresponding CPU model from the following link (note:that **Step 1** only needs to be performed once in the entire process).

2.After upgrading, the interface shows that the Boot has been downloaded successfully, as shown in Figure 2 below

 ![](../../../rk356x_img/maskrom_with_flash1.png)

3.Then click read store list, this time will see in the list are some different types of storage media, including ` X ` said device does not exist, the media ` 0 ` said the media exist, but is not selected, ` √` said the media exist and in the selected state, the next operation is based on the choice of the media,For example, if you want to operate the EMMC, you need to click the EMMC in the list first, and then click 'Switch to Storage'. At this time, the EMMC's status will go from '0' to '√', indicating that it is selected. At this time, you can go to the 'Erase All' page in Figure 1 and go back to the 'Upgrade Firmware' page to erase and burn the EMMC.

4.Nor-flash is handled the same way as EMMC, but because Nor-flash is too small to mount the system image, it is usually only erased.


5.It should be noted that if the device enters the Markrom state and chooses to upgrade the firmware directly without performing the above operation of switching the device, the following errors will occur and cause the upgrade failure, as shown in Figure 3:

 ![](../../../rk356x_img/maskrom_with_flash2.png)


6.That is because by default 3566/3568 is preferred to upgrade firmware in nor-Flash under Markrom mode. As the on-board flash is generally too small, and the system firmware is relatively large, the upgrading will fail. Therefore, in this case, in addition to the steps 1, 2 and 3 above,One more step to add is to erase the nor-flash that was upgrade by mistake (select the storage media as nor-flash and click 'Erase All') so that the device can be properly booted from EMMC instead of nor-flash after the correct firmware was upgraded.

