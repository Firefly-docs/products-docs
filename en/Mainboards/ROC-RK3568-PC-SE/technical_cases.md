# Technical Case

## SSD boot

This chapter introduces how to flash the firmware to the SSD and boot the system from the SSD.

Ready to work:
- RK356x Board
- Good M.2 SSD or SATA SSD
- Good Type-C cable
- Update to the latest Linux SDK, go to the u-boot directory, make sure to have the following commits:
```bash
commit 9341129a705816604b7883313857307f248b57e5
Author: weicz <wcz@t-chip.com.cn>
Date: Mon Mar 21 11:43:00 2022 +0800

    configs: add firefly-rk3568-usb/pcie/sata.config
    
    Signed-off-by: weicz <wcz@t-chip.com.cn>
````

### M.2 SSD boot

1. Enter the u-boot directory and execute the following commands to compile and package the boot firmware:
```bash
./make.sh firefly-rk3568-pcie --spl-new
````

The generated firmware has `rk356x_spl_loader_*.bin` and `uboot.img`.

2. RK356x enters Maskrom mode and flashes the boot firmware to Nor Flash (default):

Under Windows platform, select `Loader`, `Parameter` and `Uboot` and flash:

![](../../../rk356x_img/rkdevtool_to_nor.png)

Flash `Loader`, `Parameter` and `Uboot` under Linux platform:
```bash
upgrade_tool ul rk356x_spl_loader_*.bin
upgrade_tool di -p parameter.txt
upgrade_tool di -u uboot.img
````

3. Restart RK356x, connect the serial port, you will see the following print:
```bash
Bootdev: nvme 0
````

This means that RK356x can recognize the M.2 SSD in the uboot stage. At this time, because the M.2 SSD does not have firmware, uboot will fail to load the kernel!

4. Enter the following command in the uboot shell to flash the system firmware to the M.2 SSD:
```bash
=> rockusb 0 nvme 0
````

Note that if the device uses extboot, it needs to be canceled. Currently, M.2 SSD is not compatible with extboot.

Cancel extboot:

```diff
diff --git a/rk356x/firefly-rk3568-ubuntu.mk b/rk356x/firefly-rk3568-ubuntu.mk
index 8badc34..ee1610f 100644
--- a/rk356x/firefly-rk3568-ubuntu.mk
+++ b/rk356x/firefly-rk3568-ubuntu.mk
@@ -18,7 +18,7 @@ export RK_IDBLOCK_UPDATE=true
 # update spl
 export RK_LOADER_UPDATE_SPL=true
 #Set extboot
-export FF_EXTBOOT=true
+export FF_EXTBOOT=false
 
 # Set rootfs type, including ext2 ext4 squashfs
 export RK_ROOTFS_TYPE=ext4
```

Then flash `boot.img` again.

5. After the firmware is flashed successfully, restart and start normally:
```bash
root@firefly:~# df -h
Filesystem Size Used Avail Use% Mounted on
udev 3.8G 0 3.8G 0% /dev
tmpfs 769M 816K 768M 1% /run
/dev/nvme0n1p6 461M 363M 71M 84% /root-ro
/dev/nvme0n1p7 219G 69M 219G 1% /userdata
overlayroot 219G 69M 219G 1% /
tmpfs 3.8G 0 3.8G 0% /dev/shm
tmpfs 5.0M 4.0K 5.0M 1% /run/lock
tmpfs 3.8G 0 3.8G 0% /sys/fs/cgroup
tmpfs 769M 0 769M 0% /run/user/0
````

You can see that the device mounted on the `userdata` partition is `/dev/nvme0n1p7`!

### SATA SSD boot

The operation steps for SATA SSD booting are similar to those for M.2 SSD booting, with the following differences:

1. The command to compile and boot the firmware is as follows:
```bash
./make.sh firefly-rk3568-sata --spl-new
````

2. Enter the following command in the uboot shell to flash the system firmware to the SATA SSD:
```bash
=> rockusb 0 scsi 0
````

### Without Nor Flash

Since Maskrom does not support loading boot firmware from SSD, if there is no Nor Flash, you need to flash boot firmware to eMMC memory.

- If the eMMC memory itself has firmware, then directly enter the Loader mode to flash `rk356x_spl_loader_*.bin` and `uboot.img`!
- If the eMMC memory itself has no firmware, then Maskrom mode is programmed to the eMMC memory by default.