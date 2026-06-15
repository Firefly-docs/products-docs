# PCIe

## Introduction
There is one PCIe3.0 x 4 interface and PCIe2.0 x 2 on the EC-R3588SPC-FD development board, as shown in the figure:

On board default support for [M.2 SATA](https://wiki.t-firefly.com/en/ROC-RK3588-RT/usage_sata.html) and [PCIE WiFi/BT module](https://wiki.t-firefly.com/en/ROC-RK3588-RT/module_wireless.html)
![](../../../rk3588_img/EC-R3588SPC-FD/usage_pcie_interface.png)

## Software configuration
The available hardware resources of RK3588 PCIe and the corresponding relationship between the `pcie` controller node and PHY node on the software are shown in the figure:
![](../../../rk3588_img/EC-R3588SPC-FD/usage_pcie_phy_en.jpg)

### DTS configuration
Generally, configure the power supply pin and reset pin in DTS according to the schematic diagram, and select the correct `pcie` controller node and PHY node to enable.

There is the following configuration about [4 x 2.5G mesh EXT board]() in `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-rt-ext.dtsi`:
```
/* pcie3.0 x 4 Lane */
&pcie30phy {
    rockchip,pcie30-phymode = <PHY_MODE_PCIE_NABIBI>;
    status = "okay";
};

&pcie3x2 {
        num-lanes = <1>;
        reset-gpios = <&gpio3 RK_PD4 GPIO_ACTIVE_HIGH>;
        vpcie3v3-supply = <&vcc3v3_pcie30>;
    rockchip,skip-scan-in-resume;
    rockchip,perst-inactive-ms = <500>;
    status = "okay";
};
&pcie3x4 {
        num-lanes = <1>;
        reset-gpios = <&gpio4 RK_PB6 GPIO_ACTIVE_HIGH>;
        vpcie3v3-supply = <&vcc3v3_pcie30>;
    rockchip,skip-scan-in-resume;
    rockchip,perst-inactive-ms = <700>;
    status = "okay";
};

&vcc3v3_pcie30{
        gpios = <&gpio1 RK_PB2 GPIO_ACTIVE_HIGH>;
        regulator-always-on;
        startup-delay-us = <5000>;
        status = "okay";
};
```
`pcie30phy`：PHY node

`pcie3x2`：pcie3x2 controller node

`pcie3x4`：pcie3x4 controller node

`reset-gpios`：reset pin properties

`vcc3v3_pcie30`：power supply pin node

## Mount

### Auto mount
Format the hard drive to a usable format in the Android system interface to mount it automatically at boot

### Command to mount manually

* Find device nodes
```
ls /dev/block/nvme*                                 
/dev/block/nvme0n1
```

* Formatted as EXT4 file format
```
mkfs.ext4 /dev/block/nvme0n1
```

* Mount
```
mount /dev/block/nvme0n1 /mnt/media_rw/
```

* View the mount path
```
df -h
/dev/block/nvme0n1               916G  24K  916G   1% /mnt/media_rw
```
or
```
cat /proc/mounts  | grep nvme
/dev/block/nvme0n1 /mnt/media_rw ext4 rw,seclabel,relatime 0 0
```

## Read and write speed
The transfer rate of PCIe3.0 x 4 is theoretically 4 GB/s. You can refer to the following commands to test the read and write speed:
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
fio -filename=/dev/block/nvme0n1 -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
# Read
fio -filename=/dev/block/nvme0n1 -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
```
