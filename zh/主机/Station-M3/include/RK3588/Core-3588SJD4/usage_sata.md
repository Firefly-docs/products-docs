# SATA 使用

## 简介
ROC-RK3588S-PC 开发板上有 1 个 SATA 接口和 1 个 M.2 SATA 接口

![](../../../rk3588_img/Station-M3/usage_sata_interface.jpg)
![](../../../rk3588_img/Station-M3/usage_sata2_interface.jpg)

注意事项：

* M.2 SATA 接口，注意应选择 <font color=#ff00>SATA 协议的 SSD </font> 接入，而不是 NVME 协议的 SSD

## 软件配置
ROC-RK3588S-PC 开发板上的 1 个 SATA 接口和 1 个 M.2 SATA 接口都是 `USB3.0` 扩展出来的接口。
### DTS 配置
一般根据原理图在 DTS 中配置供电引脚，选择正确的控制器和节点使能就可以。

在 `kernel-5.10/arch/arm64/boot/dts/rockchip/aio-3588sjd4.dtsi` 中有下面一段配置：
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
`usbhost3_0 usbhost_dwc3_0`：USB3.0 控制器节点

`combphy2_psu`：PHY 节点

`vcc_3v0_sata`：M.2 SATA 供电引脚节点

## 挂载

### 自动挂载
在 Android 系统界面中将硬盘格式化为可用格式就可以开机自动挂载

### 命令手动挂载
* 查找设备节点
```
ls /dev/block/sd*                                 
/dev/block/sda
/dev/block/sda1
/dev/block/sdb
/dev/block/sdb1
```

* 格式化为EXT4文件格式
```
mkfs.ext4 /dev/block/sda1
mkfs.ext4 /dev/block/sdb1
```

* 挂载
```
mount /dev/block/sda1 /mnt/media_rw/
mount /dev/block/sdb1 /mnt/media_rw/
```

* 查看挂载路径
```
df -h
/dev/block/sda1               916G  24K  916G   1% /mnt/media_rw
```
或者
```
cat /proc/mounts  | grep sda
/dev/block/sda /mnt/media_rw ext4 rw,seclabel,relatime 0 0
```

## 读写测速
SATA2.0 的传输速率理论上达到 3.0 Gbps，可以参考如下命令进行读写速度测试：
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
fio -filename=/dev/block/sda1 -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
# 读
fio -filename=/dev/block/sdb1 -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
```

