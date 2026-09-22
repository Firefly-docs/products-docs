# FAQ

## Q: How to enter EDL (Emergency Download) mode?

**A:** There are two ways:

* Hardware way: power off the device completely, then trigger the EDL mode with the recovery (EDL) key. The key position and detailed operations differ between products, please refer to the "Enter EDL mode" section of [Upgrade Firmware](qfil_upgrade_firmware.md).
* Software way: while the device is running normally, connect the download port of the device with the PC through the data cable, then run this command on the device terminal or debug console:

```shell
sudo systemctl reboot edl
```

## Q: How to confirm that the device has entered EDL mode?

**A:** If the device entered EDL mode successfully, the QFIL tool will show "Qualcomm HS-USB QDLoader 9008". If it shows "Please Select an Existing Port", click "Select Port", you will also see Qualcomm HS-USB QDLoader 9008, select it and click "OK".

**Note: the 9008 device must be shown.** If another number is displayed (such as 900E), it means the device state is abnormal, please power off and try to enter the upgrade mode again.

## Q: QFIL shows "No Port Available" / the device cannot be found?

**A:** It is usually a driver or connection problem. Please check the following items:

* Make sure both USB drivers are installed on the Windows PC: install the **Qualcomm USB Driver** first, then install the **Google USB driver** (extract usb_driver_r13-windows, right click android_winusb.inf and choose install). Both drivers can be downloaded from the official resources download page.
* Make sure the data cable is in good condition. It is recommended to try another USB cable or another USB port of the PC.
* Enter EDL mode again and check whether the 9008 device appears in the QFIL port list.

## Q: What to do if QFIL fails during flashing?

**A:** Please troubleshoot as follows:

* Make sure the FireHose Configuration is set correctly according to the tutorial, then click "OK" again.
* Make sure the correct programmers file is selected (`xbl_s_devprg_ns.melf` or `prog_firehose_ddr.elf`, it differs between chips), and the file type is set to All Files.
* When loading XML, all the xml files need to be selected twice.
* If it still fails, power off and enter EDL mode again, change the USB cable or the USB port and flash again.

For the complete flashing steps, please refer to [Upgrade Firmware](qfil_upgrade_firmware.md).

## Q: What to do if the device does not boot normally after flashing?

**A:** After flashing, please wait a while, the device will reboot to normal mode automatically. If it does not reboot for a long time or cannot enter the system:

* Flash the complete firmware again (flashing the complete firmware will update the data of all partitions and erase all the data on the board).
* If the problem occurs after flashing a partition image, the partition and the file may not match. Please restore with the complete firmware first, and then flash partitions separately.

## Q: How to flash partitions?

**A:** The following content is intended for advanced users who need to flash partition images (such as boot, dtbo, etc.) separately. If you only need a full firmware upgrade, please refer to [Upgrade Firmware](qfil_upgrade_firmware.md). Only products that support partition image upgrade can operate in this way.

### Enter Bootloader Mode

While the device is running normally, connect the download port of the device with the PC through Type-A USB cable, then run this command on the device terminal or debug console:

```shell
sudo systemctl reboot bootloader
```

<center>

<img alt="" src="../../../qcom_img/AIBOX-9075/download_port.jpg" width="700">
</center>

Open the Windows terminal (PowerShell), navigate to the location of Android Platform Tools, then run `.\fastboot.exe devices`. If the device enters the bootloader mode successfully, fastboot will show a detected device:

```
PS D:\path\to\platform-tools> .\fastboot.exe devices
xxxxxxxx         fastboot
```

If fastboot shows nothing, please check the driver installation and the USB cable.

### Flash Partition Images

Usage: `fastboot.exe flash <partition name> <path to image>`

For example:

```
.\fastboot.exe flash boot D:\path\to\boot.img

.\fastboot.exe flash dtbo D:\path\to\dtbo.img

# reboot device after flashing.
.\fastboot.exe reboot
```

**Note: flashing a wrong file will cause system failure.** Please confirm the partition and the file before flashing; if you are not sure, please ask for help first instead of blind operation.
