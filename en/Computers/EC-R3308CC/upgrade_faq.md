# FAQ

## Q: What boot modes does the device have?

**A:** The EC-R3308CC has three boot modes:

* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the normal boot process. Each component loads in turn and the system starts normally.

### Loader mode

In Loader mode, the bootloader enters the upgrade state and waits for commands from the host computer, which is used for firmware upgrade and so on. To enter Loader mode, the bootloader must detect that the `RECOVERY` (recovery) key is pressed and the USB is connected at startup.

### MaskRom mode

MaskRom mode is used for system repair when the bootloader is damaged.

Under normal circumstances, it is not necessary to enter `MaskRom mode`. Only when the bootloader verification fails (the IDB block cannot be read, or the bootloader is damaged), the BootRom code will enter `MaskRom mode`. At this time, the BootRom code waits for the host to transfer the bootloader code through the USB interface, then loads and runs it.

## Q: How to make the device enter the upgradable mode (Loader mode)?

**A:** Connect the device and the host with the Type-C data cable, then choose one of the following methods:

* One way is to disconnect the power adapter

    * Press and hold the RECOVERY (recovery) key on the device.
    * Connect the power.
    * After about two seconds, release the RECOVERY key.

* Another way is to connect the power adapter

    * Press and hold the RECOVERY (recovery) key on the device.
    * Short-press the RESET (reset) key once.
    * After about two seconds, release the RECOVERY key.

You can also enter Loader mode by running the following command in the serial debug terminal or adb shell:

```shell
reboot loader
```

## Q: How to tell which upgrade mode the device is currently in?

**A:** To determine which upgrade mode the board is currently in, we can check it with the upgrade tool.

**Windows Operating System**

The AndroidTool/RKDevTool displays the prompt `Found One LOADER Device` at the bottom

<center>

<img alt="" src="../../../rk3308_img/upgrade_firmware_androidtool_zh.png" width="800">
</center>

If the "Enter Loader mode" operation has been performed, but the upgrade tool still does not show LOADER, check whether the Windows host prompts that new hardware is found and the driver is configured. Open the Device Manager and a new device `Rockusb Device` will appear, as shown below. If not, you can go back to the previous step to [install the driver](03-upgrade_firmware.md).

<center>

<img alt="" src="../../../rk3308_img/upgrade_firmware_new_equipment.png" width="800">
</center>

If the burning tool prompts `Found One MASKROM Device`, it means that the device is currently in MaskRom mode.

**Linux Operating System**

After running upgrade_tool, you can see a `Loader` prompt in the list of connected devices:

```shell
firefly@T-chip:~/severdir/down_firmware$ sudo upgrade_tool
List of rockusb connected
DevNo=1 Vid=0x2207,Pid=0x330c,LocationID=106    Loader
Found 1 rockusb,Select input DevNo,Rescan press <R>,Quit press <Q>:q
```

If `MaskRom` is shown in the list, it means that the device is currently in MaskRom mode.

**Debug serial port assistance**

If the device is connected to the debug serial port, you can also observe the U-Boot/kernel boot logs through the serial terminal to help determine which mode the device is in. Taking MobaXterm under Windows as an example, the serial port is configured as follows (baud rate 1500000, for the connection method please refer to [Debug Serial Port](usb_to_ttl.md)):

<center>

<img alt="" src="../../../rk3308_img/debug_set_MobaXterm1.PNG" width="800">
</center>

<center>

<img alt="" src="../../../rk3308_img/debug_set_MobaXterm2.PNG" width="800">
</center>

For how to make the device enter the upgradable mode, please refer to [Upgrade the firmware via USB cable](03-upgrade_firmware.md).

## Q: How to flash partitions?

**A:** The following content is intended for advanced users who need to flash partitions (boot/kernel/rootfs, etc.) separately. If you only need a full firmware upgrade, please refer to [Upgrade the firmware via USB cable](03-upgrade_firmware.md).

### Windows Operating System

#### Flash partition images

The steps to flash the partition images are as follows:

1. Switch to the `Upgrade Firmware` page.
2. Check the partitions to be burned, and multiple selections are allowed.
3. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.
4. Click the `Run` button to start the upgrade, and the device will restart automatically after the upgrade.

<center>

![AndroidTool upgrade tool interface](../../../rk3308_img/upgrade_firmware_androidtool_zh.png)
</center>

### Linux Operating System

#### Flash partition images

```shell
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -dtbo /path/to/dtbo.img
sudo upgrade_tool di -p paramater   #flash parameter
sudo upgrade_tool ul bootloader.bin #flash bootloader
```

If the upgrade fails due to flash problems, you can try low-level formatting and erasing the emmc:

```shell
sudo upgrade_tool lf update.img	# low-level formatting
sudo upgrade_tool ef update.img	# erase
```

## Q: What to do if an exception occurs during flashing?

**A:**

### Analysis of flashing failure

If Download Boot Fail occurs during the flashing process, or an error occurs during the flashing process, as shown in the figure below, it is usually caused by a poor connection of the USB cable, an inferior cable, or the insufficient drive capability of the USB port of the computer. Please replace the USB cable or the USB port of the computer to troubleshoot.

<center>

<img alt="" src="../../../rk3308_img/upgrade_downloadfail.png" width="800">
</center>


### Device not recognized / new device prompt

If the "Enter Loader mode" operation has been performed, but the burning tool still does not show LOADER, it is usually because the host has not recognized the device or the driver is not installed properly. The handling method has been described in the section [How to tell which upgrade mode the device is currently in?](#q-how-to-tell-which-upgrade-mode-the-device-is-currently-in) above (including the screenshots of the new device `Rockusb Device` and the driver installation), and will not be repeated here.

If the upgrade fails, you can try to erase the flash first and then upgrade. See the section [How to flash partitions?](#q-how-to-flash-partitions) above.

## Q: How to enter MaskRom mode?

**A:** `MaskRom` mode is the last line of defense when the device is bricked. Forcibly entering `MaskRom` involves hardware operations and carries certain risks. Therefore, only try `MaskRom` mode when the device cannot enter `Loader` mode. The principle of entering `MaskRom` is to manually short-circuit the data pins of the EMMC to ground. The system will consider the EMMC data to be in error and clear the EMMC data.

**Please read carefully and operate carefully!**

To enter MaskRom mode on the complete machine, you need to disassemble the host and find the corresponding test points on the motherboard for shorting. The location of the test points is subject to the actual motherboard.

