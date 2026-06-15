# SATA 使用


## 简介
ITX-3588J  开发板上有 1 个 M.2 接口

可以软件配置成 M.2 SATA3.0 接口，支持 SATA 协议的 SSD 使用，也可以软件配置成 M.2 PCIe2.0 接口，支持 NVMe 协议的 SSD 使用。

默认软件配置成 M.2 SATA3.0 接口, 支持 SATA 协议的 SSD 使用

![](../../../rk3588_img/Core-3588J/usage_sata_m2_sata.jpg)

## 软件配置
 
### DTS 配置
一般根据原理图在 DTS 中选择正确的控制器节点和 PHY 节点使能，并关闭与其复用的控制器节点就可以。

在 `kernel/arch/arm64/boot/dts/rockchip/aio-3588jd4.dtsi` 中有下面一段配置：
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
`sata0`：sata0 控制器节点

`combphy0_ps`：PHY 节点

`pcie2x1l2`：pcie2x1l2 控制器节点

`vcc_sata_pwr_en`：M.2 SATA 供电引脚节点

`M2_SATA_OR_PCIE`：默认值为 1 即配置成 SATA3.0，如果需要配置成 PCIe2.0，需修改为 0

## 挂载

### 自动挂载
在 Android 系统界面中将硬盘格式化为可用格式就可以开机自动挂载

### 命令手动挂载
* 查找设备节点
```
ls /dev/block/sd*                                 
/dev/block/sda
```

* 格式化为EXT4文件格式
```
mkfs.ext4 /dev/block/sda
```

* 挂载
```
mount /dev/block/sda /mnt/media_rw/
```

* 查看挂载路径
```
df -h
/dev/block/sda               916G  24K  916G   1% /mnt/media_rw
```
或者
```
cat /proc/mounts  | grep sda
/dev/block/sda /mnt/media_rw ext4 rw,seclabel,relatime 0 0
```

## 读写测速
SATA3.0 的传输速率理论上达到 6.0 Gbps，可以参考如下命令进行读写速度测试：
* dd
```
# 路径根据实际挂载路径修改
# 写1G文件
echo 3 > /proc/sys/vm/drop_caches
busybox dd if=/dev/zero of=/mnt/media_rw/41AD-09EA/test1 bs=1M count=1024 conv=sync
# 读1G文件
echo 3 > /proc/sys/vm/drop_caches
busybox dd if=/mnt/media_rw/41AD-09EA/test1 of=/dev/null conv=sync
```

* fio
```
# 使用 fio 会格式化硬盘
# 写
fio -filename=/dev/block/sda -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
# 读
fio -filename=/dev/block/sda -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
```

