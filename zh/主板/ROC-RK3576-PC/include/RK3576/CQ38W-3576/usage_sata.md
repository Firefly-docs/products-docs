# SATA 使用

## 简介
ROC-RK3576-PC 开发板上有 1 个 M.2 接口。

可以软件配置成 M.2 SATA3.1 接口，支持 SATA 协议的 SSD 使用，也可以软件配置成 M.2 PCIe2.1 接口，支持 NVMe 协议的 SSD 使用。

默认软件配置成 M.2 SATA3.1 接口, 支持 SATA 协议的 SSD 使用。

![](../../../rk3576_img/ROC-RK3576-PC/usage_sata_interface.jpg)

## 软件配置
<!--
### 方式一：系统设置修改

Settings->Connected devices -> M.2 SSD Type

选择需要生效的选项SATA 或 PCIe

 ![](../../../rk3576_img/ROC-RK3576-PC/swtich_sata_pcie.jpg)

 修改后需要重启系统才会生效
 -->

一般根据原理图在 DTS 中选择正确的控制器节点和 PHY 节点使能，并关闭与其复用的控制器节点就可以。

在 `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576q.dts` 以及 `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576q.dtsi`中有如下配置：
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
        reset-gpios = <&gpio4 RK_PA7 GPIO_ACTIVE_HIGH>;
};
#endif

```

`sata0`：sata0 控制器节点

`pcie0`：pcie0 控制器节点

默认配置成SATA3.1，如果需要配置成 PCIe2.1，需做如下修改，二者只能使能其一
```
#define SATA            0
#define PCIE            1

```

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

