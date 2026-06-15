# SATA

## Introduction
There are 4 SATA ports and 1 M.2 SATA port on the ITX-3588J development board.

![](../../../rk3588_img/Core-3588SG/usage_sata_interface.jpg)

![](../../../rk3588_img/Core-3588SG/usage_sata_m2_sata.jpg)

![](../../../rk3588_img/Core-3588SG/usage_sata_led.jpg)

Precautions:

* Use the official <font color=#ff00>12V/7A power adapter</font>, otherwise the SATA hard disk may not be recognized due to insufficient power supply.

* M.2 SATA interface, please select <font color=#ff00>SATA protocol SSD </font> instead of NVME protocol SSD

## Software configuration
The available hardware resources of RK3588 SATA and the corresponding relationship between the `sata` controller node and PHY node on the software are shown in the figure:
![](../../../rk3588_img/Core-3588SG/usage_sata_phy_en.jpg)
The 4 SATA ports and 1 M.2 SATA port on the ITX-3588J development board are all ports extended by the `SATA PM` expansion chip, which uses the RK3588's SATA0 group of resources.
### DTS configuration
Generally, configure the power supply pins in DTS according to the schematic diagram, select the correct `sata` controller node and PHY node to enable, and close the `pcie` controller node that is multiplexed with it.

There is the following configuration in `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dtsi`:
```
/* sata pm */
&combphy0_ps {
    status = "okay";
};

&sata0 {
    status = "okay";
};

&vcc_sata_pwr_en{
    status = "okay";
    gpio = <&pca9555 PCA_IO1_2 GPIO_ACTIVE_HIGH>;  //PCA_IO 12
};
```
`sata0`：sata0 controller node

`combphy0_ps`：PHY node

`vcc_sata_pwr_en`：M.2 SATA power pin node

In `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi`, the `pcie` controller node that is reused with it has been turned off by default:
```
pcie2x1l2: pcie@fe190000 {
    ...
    status = "disabled";
    ...
};
```
`pcie2x1l2`：`pcie` controller node multiplexed with `sata0`

## Mount

### Auto mount
Format the hard drive to a usable format in the Android system interface to mount it automatically at boot

### Command to mount manually
* Find device nodes
```
ls /dev/block/sd*                                 
/dev/block/sda
```

* Formatted as EXT4 file format
```
mkfs.ext4 /dev/block/sda
```

* Mount
```
mount /dev/block/sda /mnt/media_rw/
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
fio -filename=/dev/block/sda -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
# Read
fio -filename=/dev/block/sda -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
```

