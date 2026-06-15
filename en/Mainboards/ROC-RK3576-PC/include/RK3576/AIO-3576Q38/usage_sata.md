# SATA

## Introduction
There is 1 M.2 interface on the ROC-RK3576-PC development board.

It can be configured as an M.2 SATA3.1 interface by software for use with SSDs that support the SATA protocol, or as an M.2 PCIe2.1 interface by software to support the use of SSDs with the NVMe protocol.

The default software is configured as M.2 SATA3.1 interface, which supports the use of SSDs with SATA protocol.

![](../../../rk3576_img/ROC-RK3576-PC/usage_sata_interface.jpg)

## Software configuration
<!--
### Method 1:Modify system settings

Settings->Connected devices -> M.2 SSD Type

Select the option SATA or PCIe that needs to take effect

 ![](../../../rk3576_img/ROC-RK3576-PC/swtich_sata_pcie.jpg)

 The modification will take effect only after the system is restarted
-->

Generally, according to the schematic diagram, select the correct controller node and PHY node to enable in DTS, and close the multiplexed controller node.

There is the following configuration in `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576q38.dts` and `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576q38.dtsi`:

```
#define SATA            1
#define PCIE            0
```
```
#if SATA && ( PCIE == 0 )
&sata0 {
        status = "okay";
        pinctrl-0 = <&sata_reset>;
        pinctrl-names = "default";
};
#endif

#if PCIE && ( SATA == 0 )
&pcie0 {
        status = "okay";
        reset-gpios = <&gpio4 RK_PA5 GPIO_ACTIVE_HIGH>;
};
#endif

```

`combphy0_ps`：PHY node

`sata0`：sata0 controller node

`pcie0`：pcie2x1l2 controller node

The default configuration is SATA3.1. If you want to set to PCIe2.1, you need to modify it as follows. Only one of the two functions can be enabled
```
#define SATA            0
#define PCIE            1

```

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

