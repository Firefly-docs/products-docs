
# FAQ

## Q: What boot modes does the device support?

**A:** AIBOX-3588 has three boot modes:

* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the regular boot process, in which all components are loaded one by one and the system starts up normally.

### Loader mode

In Loader mode, the bootloader enters the upgrade state and waits for commands from the host, which is used for firmware upgrade and so on.

### MaskRom mode

MaskRom mode is used for system repair when the bootloader is corrupted.

Normally, there is no need to enter `MaskRom mode`. Only when the bootloader verification fails (the IDB block cannot be read, or the bootloader is corrupted) will the BootRom code enter `MaskRom mode`. At this point, the BootRom code waits for the host to transfer the bootloader code via the USB interface, and then loads and runs it.

***To force entry into `MaskRom mode`, please refer to the chapter [Upgrade Firmware](upgrade_firmware.md).***

## Q: How do I know which upgrade mode the device is currently in?

**A:** Before flashing the firmware, the board needs to enter an upgradeable mode (Loader mode or MaskRom mode). For details, please refer to [Entering the Upgrade Mode](upgrade_firmware.md). After entering the corresponding mode, you can confirm it with the following tools:

**Windows**

The AndroidTool tool will show the message `Found One LOADER Device` at the bottom:
<center>

<img alt="" src="../../../aibox_img/AIBOX-3588/upgrade_firmware_androidtool_zh.png" width="800">
</center>

If the flashing tool still does not show LOADER, you can check whether the Windows host prompts that new hardware is found and the driver is configured. Open Device Manager and you will see a new device `Rockusb Device` appear, as shown in the figure below.

<center>

<img alt="" src="../../../aibox_img/AIBOX-3588/upgrade_firmware_new_equipment.jpg" width="800">
</center>

**Linux**

After running upgrade_tool, you can see a `Loader` prompt among the connected devices:

```shell
firefly@T-chip:~/severdir/down_firmware$ sudo upgrade_tool
List of rockusb connected
DevNo=1 Vid=0x2207,Pid=0x330c,LocationID=106    Loader
Found 1 rockusb,Select input DevNo,Rescan press <R>,Quit press <Q>:q
```

## Q: How to flash individual partitions?

**A:** The following content is intended for advanced users who need to flash partitions (boot/kernel/rootfs, etc.) separately; for a complete firmware upgrade, please refer to [Upgrading Firmware with a USB Cable](upgrade_firmware.md).

### Windows

#### Flash Partition Images

The steps to flash partition images are as follows:

1. Switch to the `Upgrade Firmware` page.
2. Check the partitions to be flashed. Multiple selections are allowed.
3. Make sure the paths of the image files are correct. If necessary, click the blank table cell to the right of the path to reselect it.
4. Click the `Run` button to start the upgrade. The device will reboot automatically when the upgrade is finished.

<center>

![AndroidTool flashing tool interface](../../../aibox_img/AIBOX-3588/upgrade_firmware_androidtool_zh.png)
</center>

### Linux

#### Flash Partition Images

```shell
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -dtbo /path/to/dtbo.img
sudo upgrade_tool di -p paramater   # flash the parameter
sudo upgrade_tool ul bootloader.bin # flash the bootloader
```

If an error occurs during the upgrade due to flash issues, you can try low-level formatting or erasing the eMMC:

```shell
sudo upgrade_tool lf update.img	# low-level format
sudo upgrade_tool ef update.img	# erase
```

Flash the dynamic partitions with fastboot:

```shell
adb reboot fastboot # enter the bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # reboot after flashing successfully
```

## Q: What to do if an error occurs during flashing?

**A:**

### Flash Failure Analysis

If Download Boot Fail occurs during flashing, or an error occurs during the flashing process as shown in the figure below, it is usually caused by a poor USB cable connection, low-quality cables, or insufficient driving capability of the computer's USB port. Please replace the USB cable or try another USB port on the computer.

<center>

![Flashing failure message](../../../aibox_img/AIBOX-3588/upgrade_firmware_download_fail.png)
</center>

### Device Not Recognized / Abnormal Driver

After entering Loader mode, if the flashing tool never shows the `LOADER` prompt, the device is generally not recognized or the driver is abnormal: check whether the Windows host prompts that new hardware is found and the driver is configured, and open Device Manager to confirm whether `Rockusb Device` appears; if not, please go back and reinstall the RK USB driver and try again. For the screenshots and detailed instructions, see the section "How do I know which upgrade mode the device is currently in?" above.

### How to Force Entry into MaskRom Mode

If the board cannot enter Loader mode, you can try to force entry into MaskRom mode. For the operation method, see [Upgrade Firmware](upgrade_firmware.md), or refer to the section "How to enter MaskRom mode?" below.

## Q: How to enter MaskRom mode?

**A:** `MaskRom` mode is the last line of defense for a bricked device. Forcing entry into `MaskRom` involves hardware operations and carries certain risks, so the `MaskRom` mode should only be attempted when the device cannot enter `Loader` mode. The principle of entering `MaskRom` is to manually short-circuit the data pins of the EMMC to the ground, so that the system considers the EMMC data to be in error and clears the EMMC data.

**Please read carefully and operate with caution!**



