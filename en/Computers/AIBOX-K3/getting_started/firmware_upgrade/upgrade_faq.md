
# FAQ

<Accordions>

<Accordion title="Q: What flashing methods does the K3 support?" type="single" id="aibox_upgrade_faq_1">

**A:** The AIBOX-K3 supports two flashing methods:

* **USB cable flashing**: burn the firmware from the host to the board's storage via a Type-C cable. Both unified firmware and partition images are supported. The board needs to enter flashing mode first, then flash with the titanflasher tool. See [Upgrading Firmware Using a USB Cable](upgrade_firmware.md).
* **MaskRom (hardware flashing mode) flashing**: the last line of defense when the board cannot enter Loader mode; enter it by pressing the boot button while powering on. See [Upgrading Firmware Using a USB Cable](upgrade_firmware.md).

Please read the [Flashing Tool User Manual](https://spacemit.com/community/document/info?lang=en&nodepath=tools/user_guide/flasher_user_guide.md) carefully before flashing.

The titanflasher flashing tool interface:

<center>

<img alt="" src="../../../../../aibox_img/AIBOX-K3/upgrade_firmware_titanflasher_en.png" width="800">
</center>

</Accordion>

<Accordion title="Q: How to enter flashing mode (Loader)?" type="single" id="aibox_upgrade_faq_2">

**A:** Entering Loader mode via software requires operating the uboot terminal (the adb tool cannot be used), as follows:

1. Connect the board to the host with a Type-C cable, and connect the serial debugging terminal.
2. Run `reboot` in the serial debugging terminal to restart the board.
3. When the board reboots to the uboot stage, press and hold the `s` key to enter the uboot debugging terminal, then delete the extra `s` characters from the input.
4. Run `fastboot 0` in the uboot debugging terminal to enter flashing mode.
5. On the PC, in titanflasher, click: Dev Tools --> USB Download --> Scan Devices --> Select the flashing file --> Start Flashing.

> Note: at this point the titanflasher tool will flash part of the necessary firmware to the board, but not all of it. You need to press Enter again in the board's uboot terminal to continue the subsequent firmware flashing.

If the board cannot enter Loader mode, you can force entry into the hardware flashing mode; see "What are common errors?" below.

</Accordion>

<Accordion title="Q: Is partition flashing supported?" type="single" id="aibox_upgrade_faq_3">

**A:** Yes. Besides flashing the unified firmware, titanflasher also supports flashing partition images by configuring a partition file. Steps:

Dev Tools --> USB Download --> Scan Devices --> Local File --> Select the flashing file --> Reboot After Flashing --> Configure Partition File --> Start Flashing.

When configuring the partition file, you need to select the partition file:

1. `partition_4M.json`: updates the Nor Flash on the core board.
2. `partition_universal.json`: updates the UFS partition image on the core board.

<center>

<img alt="" src="../../../../../aibox_img/AIBOX-K3/upgrade_firmware_part_en.png" width="800">
</center>

</Accordion>

<Accordion title="Q: What to do if flashing fails?" type="single" id="aibox_upgrade_faq_4">

**A:** If an error occurs during flashing, it is usually caused by a poorly connected USB cable, a low-quality cable, or insufficient driving capability of the computer's USB port. Troubleshoot as follows:

1. Replace the Type-C cable with one in good condition.
2. Try another USB port on the computer, rescan the device and retry.
3. If it still fails, try forcing entry into the hardware flashing mode and flash again (see "What are common errors?" below).

</Accordion>

<Accordion title="Q: What are common errors?" type="single" id="aibox_upgrade_faq_5">

**A:**

* **Cannot enter Loader mode**: you can force entry into the hardware flashing mode: connect the PC to the board's Type-C USB 3.0 port with a Type-C cable (be careful not to connect it to the USB serial port by mistake), press and hold the boot button and then power on; the device will enter the hardware flashing mode. Then click "Scan Devices" in the flashing tool to identify the device.

<center>

<img alt="" src="../../../../../aibox_img/AIBOX-K3/upgrade_maskrom_en.png" width="800">
</center>

* **Device not recognized by the flashing tool**: make sure the Type-C cable is well connected and not plugged into the wrong port (it should be the Type-C USB 3.0 port). Replace the USB cable or the computer's USB port if necessary, and click "Scan Devices" again.
* **Nor Flash related**: `partition_4M.json` updates the Nor Flash on the core board. Before flashing, check whether a Nor Flash chip is soldered on the core board, to avoid selecting the wrong partition file.

</Accordion>

<Accordion title="Q: What boot modes does the device support?" type="single" id="aibox_upgrade_faq_6">

**A:** AIBOX-K3 has the following boot modes:

* Normal mode
* Loader mode
* Hardware flashing mode

**Normal mode**

Normal mode is the regular boot process, in which all components are loaded one by one and the system starts up normally.

**Loader mode**

In Loader mode, the bootloader enters the upgrade state and waits for commands from the host, which is used for firmware upgrade and so on.

**Hardware flashing mode**

Hardware flashing mode is used for system repair when the bootloader is corrupted.

</Accordion>

<Accordion title="Q: Can the board get bricked?" type="single" id="aibox_upgrade_faq_7">

**A:** AIBOX-K3 offers flexible boot options. Generally, unless there is hardware damage, the board will not become bricked. If an accident occurs during the upgrade process and the bootloader is damaged, making it impossible to upgrade again, you can still force the board into the hardware flashing mode to repair it.

***To force entry into the hardware flashing mode, please refer to the chapter [Upgrading Firmware Using a USB Cable](upgrade_firmware.md).***

</Accordion>

<Accordion title="Q: Which storage does the device boot from?" type="single" id="aibox_upgrade_faq_8">

**A:** AIBOX-K3 boots the system from the following storage:

* UFS interface
* PCIe interface

</Accordion>

</Accordions>


