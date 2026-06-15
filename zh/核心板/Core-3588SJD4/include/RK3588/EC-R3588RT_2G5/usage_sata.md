# SATA 使用

## 简介
Core-3588SJD4 开发板上有 1 个 M.2 接口。

默认软件配置成 M.2 SATA3.0 接口, 支持 SATA 协议的 SSD 使用。

![](../../../rk3588_img/Core-3588SJD4/usage_pcie_interface.png)

## 软件配置
### DTS 配置
一般根据原理图在 DTS 中选择正确的控制器节点和 PHY 节点使能，并关闭与其复用的控制器节点就可以。

在 `kernel/arch/arm64/boot/dts/rockchip/roc-rk3588-rt.dtsi` 中有下面一段配置：
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
`combphy2_psu`：PHY 节点

`sata2`：sata2 控制器节点

`pcie2x1l1`：pcie2x1l1 控制器节点

`vcc_sata_pwr_en`：vcc_sata_pwr_en 控制sata供电

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

