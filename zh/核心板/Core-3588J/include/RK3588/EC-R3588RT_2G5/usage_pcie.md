# PCIe 使用

## 简介
Core-3588J 开发板上有 3 个 PCIe2.0 x 3 接口 和 1 个 PCIe3.0 x 4 接口，其中 2 个 PCIe2.0 x 2 （M.2 SATA / PCIe Wifi/BT 模块），PCIe3.0 x 4 引出接口给扩展板使用，

板载默认支持[M.2 SATA](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-RT/usage_sata.html) 和 [PCIe WiFi/BT模块](https://wiki.t-firefly.com/zh_CN/ROC-RK3588-RT/module_wireless.html)。

如图：

![](../../../rk3588_img/Core-3588J/usage_pcie_interface.png)


## 软件配置
关于 RK3588 PCIe 的硬件可用资源及软件上 pcie 控制器节点、 PHY 节点对应关系如图：
![](../../../rk3588_img/Core-3588J/usage_pcie_phy.png)

Core-3588J 开发板上的 PCIe3.0 x 4 接口使用了 RK3588 的 `PCIe Gen3 x 4 lane` 和 `PCIe Gen3 x 2 lane`  这组资源。
### DTS 配置
一般根据原理图在 DTS 中配置供电引脚、复位引脚，选择正确的 pcie 控制器节点和 PHY 节点使能就可以。

示例： 

以下是一段 [4 x 2.5G 网口扩展板]() 的DTS配置 `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-rt-ext.dtsi` ：
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
`pcie30phy`：PHY 节点

`pcie3x2`：pcie3x2 控制器节点

`pcie3x4`：pcie3x4 控制器节点

`reset-gpios`：复位引脚属性

`vcc3v3_pcie30`：供电引脚节点

## 挂载

### 自动挂载
在 Android 系统界面中将硬盘格式化为可用格式就可以开机自动挂载

### 命令手动挂载

* 查找设备节点
```
ls /dev/block/nvme*                                 
/dev/block/nvme0n1
```

* 格式化为EXT4文件格式
```
mkfs.ext4 /dev/block/nvme0n1
```

* 挂载
```
mount /dev/block/nvme0n1 /mnt/media_rw/
```

* 查看挂载路径
```
df -h
/dev/block/nvme0n1               916G  24K  916G   1% /mnt/media_rw
```
或者
```
cat /proc/mounts  | grep nvme
/dev/block/nvme0n1 /mnt/media_rw ext4 rw,seclabel,relatime 0 0
```

## 读写测速
PCIe3.0 x 4 的传输速率理论上达到 4 GB/s，可以参考如下命令进行读写速度测试：
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
fio -filename=/dev/block/nvme0n1 -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
# 读
fio -filename=/dev/block/nvme0n1 -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=1M -size=200G -numjobs=30 -runtime=60 -group_reporting -name=mytes
```
