# FAQ

## Q: What boot modes does the device have?

**A:** The ROC-RK3566-PC has three boot modes:

* Normal mode
* Loader mode
* MaskRom mode

### Normal mode

Normal mode is the normal boot process. Each component loads in turn and the system starts normally.

### Loader mode

In Loader mode, the bootloader enters the upgrade state and waits for commands from the host computer, which is used for firmware upgrade and so on.

### MaskRom mode

MaskRom mode is used for system repair when the bootloader is damaged.

Under normal circumstances, it is not necessary to enter `MaskRom mode`. Only when the bootloader verification fails (the IDB block cannot be read, or the bootloader is damaged), the BootRom code will enter `MaskRom mode`. At this time, the BootRom code waits for the host to transfer the bootloader code through the USB interface, then loads and runs it.

## Q: How to tell which upgrade mode the device is currently in?

**A:** To determine which upgrade mode the board is currently in, we can check it with the upgrade tool.

**Windows Operating System**

The AndroidTool displays the prompt `Found One LOADER Device` at the bottom
<center>

<img alt="" src="../../../rk356x_img/upgrade_firmware_androidtool_zh.png" width="800">
</center>

If the "Enter Loader mode" operation has been performed, but the upgrade tool still does not show LOADER, check whether the Windows host prompts that new hardware is found and the driver is configured. Open the Device Manager and a new device `Rockusb Device` will appear, as shown below. If not, you can go back to the previous step to [install the driver](03-upgrade_firmware.md).

<center>

<img alt="" src="../../../rk356x_img/upgrade_firmware_new_equipment.png" width="800">
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

For how to make the device enter the upgradable mode, please refer to [Enter Upgrade mode](03-upgrade_firmware.md).

## Q: How to flash partitions?

**A:** The following content is intended for advanced users who need to flash partitions (boot/kernel/rootfs, etc.) separately. If you only need a full firmware upgrade, please refer to [Upgrade the firmware via USB cable](03-upgrade_firmware.md).

Notice: **Linux SDK v1.2.4a** and later uses extboot, please use extboot.img instead of boot.img in the following instructions (Linux only, ignore it if using Android)

How to check SDK version:
1. The version format is vx.x.xx, eg: v1.2.4a
1. Firmware filename has SDK version(..._vx.x.xx_date.img)
1. Buildroot use`cat /etc/version`to get version(rk356x_linux_release_date_vx.x.xx.xml)
1. Ubuntu use`ffgo version`to get(rk356x_linux_release_date_vx.x.xx.xml)
1. In SDK check the link:`ls -l .repo/manifests/rk356x_linux_release.xml`
1. If you can't get version by methods above, that means you are using old version, no support for extboot

**Do not burn extboot.img into old version firmware!**

Besides, extboot ubuntu supports updating the kernel by deb package, please see [Ubuntu Manual](/en/docs/software/os-guide/Ubuntu-Debian/ubuntu-debian)

### Windows Operating System

#### Flash partition images

The steps to flash the partition images are as follows:

1. Switch to the `Upgrade Firmware` page.
2. Click the device partition table button (Dev Partition), check the partitions to be burned, and multiple selections are allowed.
3. Make sure the path of the image file is correct. If necessary, click the blank table cell on the right side of the path to select it again.
4. Click the `Run` button to start the upgrade, and the device will restart automatically after the upgrade.

<center>

![AndroidTool upgrade tool interface](../../../rk356x_img/upgrade_firmware_androidtool_zh.png)
</center>

### Linux Operating System

#### Flash the unified firmware

```shell
sudo upgrade_tool uf update.img
```

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

#### Android fastboot

Download [Linux_adb_fastboot](https://community.t-firefly.com/en/doc/download/106#other_536), and install it into the system as follows for easy invocation:

```
sudo mv adb /usr/local/bin
sudo chown root:root /usr/local/bin/adb
sudo chmod a+x /usr/local/bin/adb
```
```
sudo mv fastboot /usr/local/bin
sudo chown root:root /usr/local/bin/fastboot
sudo chmod a+x /usr/local/bin/fastboot
```

**fastboot burn dynamic partitions**

```
adb reboot fastboot # enter bootloader
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot # After the burn is successful, restart
```

## Q: What to do if an exception occurs during flashing?

**A:**

### Analysis of flashing failure

If Download Boot Fail occurs during the flashing process, or an error occurs during the flashing process, as shown in the figure below, it is usually caused by a poor connection of the USB cable, an inferior cable, or the insufficient drive capability of the USB port of the computer. Please replace the USB cable or the USB port of the computer to troubleshoot.
<center>

<img alt="" src="../../../rk356x_img/upgrade_downloadfail.png" width="800">
</center>

### Flashing exception on devices with Spi Flash (Nor Flash)

If the board is equipped with both Spi Flash (Nor Flash) and eMMC, after entering MaskRom you need to switch the upgrade storage before flashing. See [Switch the upgrade storage device](03-upgrade_firmware_with_flash.md) for details.

### Device not recognized / new device prompt

If the "Enter Loader mode" operation has been performed, but the burning tool still does not show LOADER, it is usually because the host has not recognized the device or the driver is not installed properly. The handling method has been described in the section "How to tell which upgrade mode the device is currently in?" above (including the screenshots of the new device `Rockusb Device` and the driver installation), and will not be repeated here.

## Q: How to enter MaskRom mode?

**A:** `MaskRom` mode is the last line of defense when the device is bricked. Forcibly entering `MaskRom` involves hardware operations and carries certain risks. Therefore, only try `MaskRom` mode when the device cannot enter `Loader` mode. The principle of entering `MaskRom` is to manually short-circuit the data pins of the EMMC to ground. The system will consider the EMMC data to be in error and clear the EMMC data.

**Please read carefully and operate carefully!**

