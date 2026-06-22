## 网络启动

网络启动，是用 TFTP 在服务器下载内核、dtb 文件到目标机的内存中，同时可以用 NFS 挂载网络根文件系统到目标机上，实现目标机的无盘启动。以下基于 Firefly-RK3399 板卡作出一个示例，提供用户参考。

准备工作：
* Firefly-RK3399 板卡
* 路由器、网线
* 安装有 NFS 和 TFTP 的服务器
* 一份制作好的根文件系统

注：示例中使用的是 Ubuntu 系统的 PC 机作为服务器，通过路由器和网线实现与设备的连接。用户可以根据自己的实际情况作调整，但如果是 PC 机直连设备，请使用交叉网线。请确保服务器和目标机在同一局域网内。

### 服务器部署

1、在服务器上部署 TFTP 服务：

安装 TFTP 服务：

```bash
sudo apt-get install tftpd-hpa
```

创建 `/tftpboot` 目录并赋予权限：

```bash
mkdir /tftpboot
sudo chmod 777 /tftpboot
```

然后修改 TFTP 服务器的配置文件 `/etc/default/tftpd-hpa`，修改内容为：

```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"   # 这个根据用户实际的 tftp 目录设置
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-c -s -l"
```

退出修改后重启 TFTP 服务器：

```bash
sudo service tftpd-hpa restart
```

在 `/tftpboot` 目录中创建一个文件，测试 TFTP 服务是否可用：

```bash
firefly@Desktop:~$ cd /tftpboot/
firefly@Desktop:/tftpboot$ touch test
firefly@Desktop:/tftpboot$ cd /tmp
firefly@Desktop:/tmp$ tftp 127.0.0.1
tftp> get test
tftp> q
```

查看 `/tmp` 目录中的文件，如果看到了 `test` 文件表示 TFTP 服务器是可用的。

2、在服务器上部署 NFS 服务：

安装 NFS 服务器：

```bash
sudo apt-get install nfs-kernel-server
```

创建共享目录：

```bash
sudo mkdir /nfs
sudo chmod 777 /nfs
cd /nfs
sudo mkdir rootfs
sudo chmod 777 rootfs
```

然后将定制好的根文件系统复制到 `/nfs/rootfs` 目录中。

接着配置 NFS，在 `/etc/exports` 中添加共享目录位置：

```bash
/nfs/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

共享的目录根据用户实际情况设置，其中的 "*" 代表的是所有用户可访问。

重启 NFS 服务器：

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

本地挂载共享目录，测试 NFS 服务器是否可用：

```bash
sudo mount -t nfs 127.0.0.1:/nfs/rootfs/ /mnt
```

在 `/mnt` 目录下查看到了与 `/nfs/rootfs` 一致的内容，则说明 NFS 服务器部署成功。然后可以解除挂载：

```bash
sudo umount /mnt
```

### 内核的配置

如果要做到挂载网络根文件系统，需要在内核中做相关配置，并在 dts 中修改相关挂载根文件系统的配置。

首先进行内核配置，在内核目录中执行 `make menuconfig`，选择相关配置：

```
[*] Networking support  --->
         Networking options  --->
                [*]   IP: kernel level autoconfiguration
                [*]     IP: DHCP support
                [*]     IP: BOOTP support
                [*]     IP: RARP support


File systems  --->
        [*] Network File Systems  --->
                [*]   Root file system on NFS
```

修改 `rk3399-firefly.dts` 配置，在 dts 文件中修改 `chosen` 节点下的 `bootargs` 参数，选择使用 NFS 挂载远程根文件系统，内容如下。

```
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 root=/dev/nfs rootfstype=ext4 rootwait";
};

#主要是修改 root 的值，root=/dev/nfs 表示通过 NFS 挂载网络根文件系统
```

编译内核：

```bash
make ARCH=arm64 rk3399-firefly.img -j12
```

编译完成后，将编译好的内核文件 `boot.img` 和 `rk3399-firefly.dtb` 文件复制到 `/tftpboot` 目录中：

```bash
cp boot.img /tftpboot
cp arch/arm64/boot/dts/rockchip/rk3399-firefly.dtb /tftpboot
```

详细说明可以参考内核目录中的 `kernel/Documentation/filesystems/nfs/nfsroot.txt`


### U-Boot 设置

请先确保目标机网线已插入，接入到服务器的局域网内。

目标机启动进入 U-Boot 命令行模式，设置以下参数：

```
=> setenv ipaddr 192.168.31.101     # 设置目标机 IP 地址
=> setenv serverip 192.168.31.106   # 设置 serverip 为服务器 IP 地址

# 设置从 TFTP 下载内核和 dtb 文件到相应地址，用户请根据自己实际的目标机修改相应地址
=> setenv bootcmd tftpboot 0x0027f800 boot.img \; tftpboot 0x08300000 rk3399-firefly.dtb \; bootm 0x0027f800 - 0x08300000

# 设置挂载网络根文件系统，IP 参数依次为：目标机 IP:服务器 IP:网关:网络掩码:设备名:off，可以更简单的设置 ip=dhcp，通过 DHXP 自动分配 IP
=> setenv bootargs root=/dev/nfs rw nfsroot=192.168.31.106:/nfs/rootfs,v3 ip=192.168.31.101:192.168.31.106:192.168.31.1:255.255.255.0::eth0:off

# 如果不想每次开机都设置一遍，可以保存上面配置的变量
=> saveenv
Saving Environment to MMC...
Writing to MMC(0)... done

# 启动目标机
=> boot
ethernet@fe300000 Waiting for PHY auto negotiation to complete. done
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename 'boot.img'.
Load address: 0x27f800
Loading: #################################################################
         #################################################################
         #################################################################
         #################################################################
         #################################################################
         ##################################################
         475.6 KiB/s
done
Bytes transferred = 20072448 (1324800 hex)
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename 'rk3399-firefly.dtb'.
Load address: 0x8300000
Loading: #######
         645.5 KiB/s
done
Bytes transferred = 97212 (17bbc hex)
## Booting Android Image at 0x0027f800 ...
Kernel load addr 0x00280000 size 19377 KiB
## Flattened Device Tree blob at 08300000
   Booting using the fdt blob at 0x8300000
   XIP Kernel Image ... OK
   Loading Device Tree to 0000000073edc000, end 0000000073ef6bbb ... OK
Adding bank: 0x00200000 - 0x08400000 (size: 0x08200000)
Adding bank: 0x0a200000 - 0x80000000 (size: 0x75e00000)
Total: 912260.463 ms

Starting kernel ...

...
```

在开机内核日志中可见：

```
[   12.146297] VFS: Mounted root (nfs filesystem) on device 0:16.
```

说明已经挂载上了网络根文件系统。

注意事项

*   确保 TFTP 服务器、NFS 服务器可用;
*   确保目标机先插入网线后在开机，且和服务器在同一局域网内，如果是直连目标机和服务器，请使用交叉网线;
*   内核配置中，`Root file system on NFS` 依赖于 `IP: kernel level autoconfiguration` 选项，请先选择 `IP: kernel level autoconfiguration`，之后才可以找到 `Root file system on NFS` 选项;
*   在 U-Boot 命令行中，请确认 `boot.img` 烧录地址和 dtb 文件烧录地址。（提示：`boot.img` 的文件结构中，开头有2k的头文件，然后才是 kernel。所以在 TFTP 下载内核到目标机时，要下载到对应 kernel 地址减去 0x0800 的地址上）;
*   在设置挂载远程根文件系统时，`nfsroot=192.168.31.106:/nfs/rootfs,v3` 中的 `v3` 代表 NFS 版本信息，请添加上以避免出现挂载不成功的问题。