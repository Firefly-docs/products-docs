# SATA 使用

## 简介
ITX-3588J 开发板上有 4 个 SATA 接口和 1 个 M.2 SATA 接口

![](../../../rk3588_img/ROC-RK3588S-PC/usage_sata_interface.jpg)

![](../../../rk3588_img/ROC-RK3588S-PC/usage_sata_m2_sata.jpg)

![](../../../rk3588_img/ROC-RK3588S-PC/usage_sata_led.jpg)

注意事项：

* 使用官方 <font color=#ff00>12V/7A 电源适配器</font>，否则可能因为供电不足导致无法识别 SATA 硬盘。

* M.2 SATA 接口，注意应选择 <font color=#ff00>SATA 协议的 SSD </font> 接入，而不是 NVME 协议的 SSD

## 软件配置
关于 RK3588 SATA 的硬件可用资源及软件上 sata 控制器节点、 PHY 节点对应关系如图：
![](../../../rk3588_img/ROC-RK3588S-PC/usage_sata_phy.png)
ITX-3588J 开发板上的 4 个 SATA 接口和 1 个 M.2 SATA 接口都是 `SATA PM` 扩展芯片扩展出来的接口，`SATA PM` 扩展芯片使用了 RK3588 的 SATA0 这组资源。
### DTS 配置
一般根据原理图在 DTS 中配置供电引脚，选择正确的 sata 控制器节点和 PHY 节点使能，并关闭与其复用的 pcie 控制器节点就可以。

在 `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-itx-3588j.dtsi` 中有下面一段配置：
```
/* sata pm */
&combphy0_ps {
    status = "okay";
};

&sata0 {
    status = "okay";
};

&vcc_sata_pwr_en{
    status = "okay";
    gpio = <&pca9555 PCA_IO1_2 GPIO_ACTIVE_HIGH>;  //PCA_IO 12
};
```
`sata0`：sata0 控制器节点

`combphy0_ps`：PHY 节点

`vcc_sata_pwr_en`：M.2 SATA 供电引脚节点

在 `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s.dtsi` 中已经默认关闭与其复用的 pcie 控制器节点：
```
pcie2x1l2: pcie@fe190000 {
    ...
    status = "disabled";
    ...
};
```
`pcie2x1l2`：与 `sata0` 相互复用的 pcie 控制器节点

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

