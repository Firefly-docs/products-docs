# Update Firmware

## Preparation before upgrading

PC OS: Ubuntu22.04, support for NFS services is required, and there may be some missing commands during the upgrade process, such as' sshpass', which users need to install themselves.

## Put EC-ThorT5000 into Force Recovery Mode
* Ensure that EC-ThorT5000 is powered off
* Connect EC-ThorT5000 and PC using TypeC
* Press and hold down EC-ThorT5000 Recovery-Key
* EC-ThorT5000 power on
* Release EC-ThorT5000 Recovery-Key
* Check EC-ThorT5000 Recovery Mode
    * use `lsusb` comand, EC-ThorT5000 is in Force Recovery Mode if you see the message: `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.`
        * `<bbb>` any three-digit number
        * `<ddd>` any three-digit number
        * `<nnnn>` four-digit number
            * `7026` : Jetson T5000 (P3834-0008 with 128GB)


## R38.4 (JetPack 7.1)
### Download Firmware

You can directly download it from Firefly [Download Page](https://community.t-firefly.com/en/doc/download/358)

After downloading, perform tar decompression:

```
tar xf flashImage.tar.gz
```

After decompression, enter the firmware package for upgrading

```
# enter the firmware package
cd Linux_for_Tegra
# update firmware
sudo ./l4t_initrd_flash.sh firefly-aio-thor-t5000 internal
```

If everything goes smoothly, after a successful upgrade, the following fields will appear to indicate that the upgrade is successful:
```
Copied 16896 bytes from /mnt/internal/gpt_secondary_3_0.bin to address 0x03ffbe00 in flash
[ 233]: l4t_flash_from_kernel: Successfully flash the qspi
[ 233]: l4t_flash_from_kernel: Flashing success
[ 233]: l4t_flash_from_kernel: The device size indicated in the partition layout xml is smaller than the actual size. This utility will try to fix the GPT.
Flash is successful
Reboot device
Cleaning up...
Log is saved to Linux_for_Tegra/initrdlog/flash_1-2_0_20250527-153418.log
```