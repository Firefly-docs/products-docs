# SATA

## Introduction
There is 1 M.2 interface on the EC-A3588Q development board.

The default software is configured as M.2 SATA3.0 interface, which supports the use of SSDs with SATA protocol.

![](../../../rk3588_img/EC-A3588Q/usage_pcie_interface.png)

## Software configuration
### DTS configuration
Generally, according to the schematic diagram, select the correct controller node and PHY node to enable in DTS, and close the multiplexed controller node.

There is the following configuration in `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-rt.dtsi`:
```
/* SATA */
&sata2 {
    status = "okay";
};

&combphy2_psu {
    status = "okay";
};

&pcie2x1l1{
    status = "disabled";
};

&vcc_sata_pwr_en{
    status = "okay";
    gpio = <&gpio2 RK_PB5 GPIO_ACTIVE_HIGH>;
    regulator-min-microvolt = <3300000>;
    regulator-max-microvolt = <3300000>;
    regulator-always-on;
    enable-active-high;
    startup-delay-us = <5000>;
    vin-supply = <&vcc12v_dcin>;
};
```
`combphy2_psu`：PHY node

`sata2`：sata2 controller node

`pcie2x1l1`：pcie2x1l1 controller node

`vcc_sata_pwr_en`：sata power enable node

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

