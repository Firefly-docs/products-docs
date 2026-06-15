# SATA

## Introduction
There are 1 SATA ports and 1 M.2 SATA port on the AIO-3588JQ  development board.

![](../../../rk3588_img/iCore-3588JQ/usage_sata_interface.jpg)
![](../../../rk3588_img/iCore-3588JQ/usage_sata2_interface.jpg)

Precautions:

* M.2 SATA interface, please select <font color=#ff00>SATA protocol SSD </font> instead of NVME protocol SSD

## Software configuration
The 1 SATA ports and 1 M.2 SATA port on the AIO-3588JQ  development board are all ports extended by the `USB3.0` expansion chip.
### DTS configuration
Generally, configure the power supply pins in DTS according to the schematic diagram, select the correct controller node and PHY node to enable.

There is the following configuration in `kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dtsi`:
```
    vcc_3v0_sata: vcc-3v0-sata-regulator {
        compatible = "regulator-fixed";
        regulator-name = "vcc_3v0_sata";
        regulator-always-on;
        enable-active-high;
        status = "okay";
        gpio = <&pca9555 PCA_IO0_0 GPIO_ACTIVE_HIGH>;
    };

&combphy2_psu {
        status = "okay";
};

&usbhost3_0 {
        status = "okay";
};

&usbhost_dwc3_0 {
        status = "okay";
        dr_mode = "host";
};


&vcc5v0_host {
        status = "okay";
        /delete-property/ regulator-boot-on;
        gpio = <&pca9555 PCA_IO0_1 GPIO_ACTIVE_HIGH>;
        /delete-property/ pinctrl-names;
        /delete-property/ pinctrl-0;
};
```
`usbhost3_0 usbhost_dwc3_0`：USB3.0 controller node

`combphy2_ps`：PHY node

`vcc_3v0_sata`：M.2 SATA power pin node

## Mount

### Auto mount
Format the hard drive to a usable format in the Android system interface to mount it automatically at boot

### Command to mount manually
* Find device nodes
```
ls /dev/block/sd*                                 
/dev/block/sda
/dev/block/sda1
/dev/block/sdb
/dev/block/sdb1
```

* Formatted as EXT4 file format
```
mkfs.ext4 /dev/block/sda1
mkfs.ext4 /dev/block/sdb1
```

* Mount
```
mount /dev/block/sda1 /mnt/media_rw/
mount /dev/block/sdb1 /mnt/media_rw/
```

* View the mount path
```
df -h
/dev/block/sda               916G  24K  916G   1% /mnt/media_rw
```
or
```
cat /proc/mounts  | grep sda
/dev/block/sda /mnt/media_rw ext4 rw,seclabel,relatime 0 0
```

## Read and write speed
The transfer rate of SATA3.0 is theoretically 6.0 Gbps. You can refer to the following commands to test the read and write speed:
* dd
```
# The path is modified according to the actual mount path
# Write 1G file
echo 3 > /proc/sys/vm/drop_caches
busybox dd if=/dev/zero of=/mnt/media_rw/41AD-09EA/test1 bs=1M count=1024 conv=sync
# Read 1G file
echo 3 > /proc/sys/vm/drop_caches
busybox dd if=/mnt/media_rw/41AD-09EA/test1 of=/dev/null conv=sync
```

* fio
```
# Using fio will format the hard drive
# Write
fio -filename=/dev/block/sda1 -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
# Read
fio -filename=/dev/block/sdb1 -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
```

