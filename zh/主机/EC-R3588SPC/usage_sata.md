# SATA 使用

<font color=#FF0000>注意：EC-R3588SPC默认配置是不包含SATA 或PCIe存储设备</font>

## 简介
EC-R3588SPC 上有 1 个 M.2 接口。

可以软件配置成 M.2 SATA3.0 接口，支持 SATA 协议的 SSD 使用，也可以软件配置成 M.2 PCIe2.0 接口，支持 NVMe 协议的 SSD 使用。

默认软件配置成 M.2 SATA3.0 接口, 支持 SATA 协议的 SSD 使用。

![](../../../rk3588_img/EC-R3588SPC/usage_sata_interface.jpg)

## 软件配置
### 方式一：系统设置修改

Settings->Connected devices -> M.2 SSD Type

选择需要生效的选项SATA 或 PCIe

 ![](../../../rk3588_img/EC-R3588SPC/swtich_sata_pcie.jpg)

 修改后需要重启系统才会生效
 
### 方式二：DTS 配置
一般根据原理图在 DTS 中选择正确的控制器节点和 PHY 节点使能，并关闭与其复用的控制器节点就可以。

在 `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi` 中有下面一段配置：
```
#define M2_SATA_OR_PCIE 1 /*1 = SATA , 0 = PCIe */

/* default use sata3.0 , pcie2.0 optional*/
&combphy0_ps {
    status = "okay";
};

#if M2_SATA_OR_PCIE
&sata0 {
    pinctrl-names = "default";
    pinctrl-0 = <&sata_reset>;
    status = "okay";
};
#else
&pcie2x1l2 {
    reset-gpios = <&gpio3 RK_PD1 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie20>;
    status = "okay";
};
#endif
```
`combphy0_ps`：PHY 节点

`sata0`：sata0 控制器节点

`pcie2x1l2`：pcie2x1l2 控制器节点

`M2_SATA_OR_PCIE`：**默认值为 1 即配置成 SATA3.0，如果需要配置成 PCIe2.0，需修改为 0**

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

