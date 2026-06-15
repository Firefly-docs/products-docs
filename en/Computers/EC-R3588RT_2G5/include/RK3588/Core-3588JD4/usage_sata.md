# SATA

## Introduction
There is a M.2 SATA port on the EC-R3588RT_2G5 development board.

The software can be configured as M.2 SATA3.0  to support SATA SSD , or it can be configured as M.2 PCIe2.0 to support NVMe SSD .

The default software configuration is M.2 SATA3.0, which supports the SATA SSD .

![](../../../rk3588_img/EC-R3588RT_2G5/usage_sata_m2_sata.jpg)

### DTS configuration
Generally, configure the power supply pins in DTS according to the schematic diagram, select the correct  controller node and PHY node to enable, and close the controller node that is multiplexed with it.

There is the following configuration in `kernel/arch/arm64/boot/dts/rockchip/aio-3588jd4.dtsi`:
```
#define rk3588
#include "rk3588-firefly-port-aio-3588jd4.dtsi"
#include "rk3588-diff-aio-3588jd4.dtsi"


#define M2_SATA_OR_PCIE 1 /*1 = SATA , 0 = PCIe */

 / {
/* led */
        firefly_leds: leds {
                power_led: power {
                        status = "disabled";

                        gpios = <&gpio0 RK_PC5 GPIO_ACTIVE_HIGH>;//green led
            default-state = "on";
                };

                user_led: user {
                        gpios = <&pca9555 PCA_IO0_3 GPIO_ACTIVE_HIGH>; //yellow led
            default-state = "off";
                };
        };


```
`sata0`：sata0 controller node

`combphy0_ps`：PHY node

`pcie2x1l2`：pcie2x1l2 controller node

`vcc_sata_pwr_en`：M.2 SATA power pin node

`M2_SATA_OR_PCIE`：The default value is 1, which is configured as SATA3.0. If it needs to be configured as PCIe2.0, it needs to be modified to 0.

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

