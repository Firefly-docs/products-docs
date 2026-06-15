# SATA

## Introduction
There is one M.2 SATA interface on the development board AIO-3588SJD4-AI 

It can be configured as an M.2 SATA3.0 interface by software for use with SSDs that support the SATA protocol, or as an M.2 PCIe2.0 interface by software to support the use of SSDs with the NVMe protocol.

The default software is configured as M.2 SATA3.0 interface, which supports the use of SSDs with SATA protocol.

![](../../../rk3588_img/Core-3588SJD4-AI/usage_sata_interface.png)

## Software configuration
### Method 1:Modify system settings

Settings->Connected devices -> M.2 SSD Type

Select the option SATA or PCIe that needs to take effect

 ![](../../../rk3588_img/Core-3588SJD4-AI/swtich_sata_pcie.jpg)

 The modification will take effect only after the system is restarted

### Method 2: DTS configuration
Generally, according to the schematic diagram, select the correct controller node and PHY node to enable in DTS, and close the multiplexed controller node.

There is the following configuration in `kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588l.dtsi`:
```
#define M2_SATA_OR_PCIE 1 /*1 = SATA , 0 = PCIe */

/* default use sata3.0 , pcie2.0 optional*/
/* sata pm */
&combphy0_ps {
    status = "okay";
};

#if M2_SATA_OR_PCIE
&sata0 {
    status = "okay";
};
#else
//pcie@fe190000
&pcie2x1l2 {
    reset-gpios = <&gpio4 RK_PA2 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc_sata_pwr_en>;
    status = "okay";
};
#endif
```
`combphy0_ps`：PHY node

`sata0`：sata0 controller node

`pcie2x1l2`：pcie2x1l2 controller node

`M2_SATA_OR_PCIE`：**The default value is 1, which means it is configured as SATA3.0. If it needs to be configured as PCIe2.0, it needs to be changed to 0**

## Mount

### Auto mount
Format the hard drive to a usable format in the Android system interface to mount it automatically at boot.

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

* mount
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

