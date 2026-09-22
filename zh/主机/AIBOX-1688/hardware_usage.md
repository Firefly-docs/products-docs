# 硬件功能使用

## 调试串口

AIBOX-1688 采用板载 Type-C 调试串口，使用 Type-C 转 USB 数据线连接机器的调试 Type-C 口和 PC 的 USB 口即可直连调试，无需拨码或其它开关操作。

串口参数为：

* 波特率：115200
* 数据位：8
* 停止位：1
* 奇偶校验：无
* 流控：无

登录终端的用户名、密码均为 `linaro`。串口的连接方法以及 Windows（MobaXterm）、Linux（minicom）下的使用方法详见：[调试串口](debug.md)。

## 以太网

AIBOX-1688 背面提供 2 个千兆以太网口：

* 网口 0（靠近电源接口）：默认 DHCP 动态获取 IP
* 网口 1（靠近 USB 接口）：默认静态 IP `192.168.150.1`，子网掩码 `255.255.255.0`

初次使用可将 PC 设置成同网段 IP（如 `192.168.150.2/24`），确认能 `ping` 通后使用 `ssh linaro@192.168.150.1` 远程登录（用户名、密码均为 `linaro`）。

### 网络 IP 配置（netplan）

在 Ubuntu 系统，双网口中的 `eth0` 默认是动态获取 IP 的（即 DHCP），`eth1` 是固定成了 `192.168.150.1`。

网络配置是通过 `/etc/netplan/01-netcfg.yaml` 文件完成的，这个文件可对 Ubuntu 系统原始网络配置进行修改。

```shell
linaro@sophon:~$ cat /etc/netplan/01-netcfg.yaml
network:
        version: 2
        renderer: networkd
        ethernets:
                eth0:
                        dhcp4: yes
                        addresses: []
                        optional: yes
                        dhcp-identifier: mac
                eth1:
                        dhcp4: no
                        addresses: [192.168.150.1/24]
                        optional: yes
```

如果用户删掉这个文件，那么 `eth1` 会变成跟 `eth0` 一样的 DHCP 方式获取 IP；如果用户想要把 `eth0` 也固定 IP，就只需在 `/etc/netplan/01-netcfg.yaml` 文件中修改 `eth0` 节点（参考 `eth1`），重启生效即可。

## 显示接口

AIBOX-1688 背面提供 1 路 HDMI 2.0 输出，最高支持 4K@60fps。

可以使用以下命令测试 HDMI 接口：

```shell
insmod /mnt/system/ko/soph_drm.ko # 安装显示框架驱动
systemctl stop SophonHDMI.service # 关闭 HDMI 的显示界面
systemctl restart SophonHDMI.service  # 恢复 HDMI 的显示界面
```

也可以通过 `modetest` 去测试：

```shell
modetest -M cvitek -s 42@40:1920x1080-60@RG24
```

其中 `42@40:1920x1080-60@RG24` 根据实际接的显示器的分辨率来修改。

## USB 接口

AIBOX-1688 背面提供 2 个 USB 3.0 接口（上面接口默认只支持 USB 3.0，不支持 USB 2.0），正面提供 1 个 TYPE-C USB 2.0 OTG 接口。

插入 U 盘等存储设备后，存储设备会被识别为 `/dev/sdb1` 之类的节点。系统不支持自动挂载，需要手工进行 `mount` 挂载：

```shell
# 创建挂载目录
mkdir disk
# 挂载
sudo mount /dev/sdb1 disk
# 查看 U 盘内的文件
ls disk
```

完成数据写入后，请及时使用 `sync` 或 `umount` 操作，关机时请使用 `sudo poweroff` 命令，避免暴力下电关机，以免数据丢失。

## TF 卡

AIBOX-1688 正面提供 TF 卡座（支持高速卡）。TF 卡的挂载方法与 U 盘类似：

```shell
# 创建挂载目录
mkdir media
# 挂载
sudo mount /dev/mmcblk1p1 media
# 查看 TF 卡内的文件
ls media
```

TF 卡同时也可用于固件升级，请参阅[固件升级](fw_upgrade.md)。

## SSD

SSD 在盒子内部，如果出厂没有配置，则需要自己添加；出厂固件不支持 SATA SSD，如需支持，则需要替换 Linux 内核。

一般可挂载的 PCIE 节点为：`/dev/nvme0n1p1`。

```shell
# 创建挂载目录
mkdir nvme_dir
# 挂载
sudo mount /dev/nvme0n1p1 nvme_dir
# 查看 PCIE SSD 的文件
ls nvme_dir
```

## 风扇

AIBOX-1688 风扇工作分为 4 个等级：

| 主控当前温度 | 风扇工作等级 |
|:----------:|:----------:|
| ≥ 33℃ | 1 |
| ≥ 45℃ | 2 |
| ≥ 55℃ | 3 |
| ≥ 60℃ | 4 |

风扇工作等级越高，转速越快，如果主控当前温度低于 33℃，风扇默认是关闭状态。

用户可通过执行以下命令查看当前主控的工作温度：

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

例如以下是 45.5 ℃：

```shell
linaro@bm1688:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

另外，用户也可以自己手动打开风扇（写入 0-4， 0 是关闭， 4 是最大等级）：

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```

## 内存分配调整

AIBOX-1688 的内存有一部分被分配于 NPU、VPP 和 VPU 设备，因此使用 `free -h` 等指令获取的内存数据与机器的实际内存不一致。

如需查看详细的内存分配设置，或者默认内存分配与实际需求冲突，需调整内存分配的，可参照以下步骤进行相应操作。

### 安装内存分配工具

从 [下载中心](https://community.t-firefly.com/doc/download/266) 获取内存分配工具压缩包，并将其传输到 AIBOX-1688 上，执行以下命令解压压缩包：

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

### 查看内存分配设置

在完成压缩包解压后，执行以下命令查看内存分配设置：

```bash
cd memory_edit
./memory_edit.sh -p
```

内存分配设置结果示例如下，其中 `Info: get max memory size ...` 显示的结果是每个设备可配置的最大内存，`Info: get now memory size ...` 显示的结果是每个设备目前所分配的内存

```bash
linaro@aibox-1688x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1688x_se7_v1_mini.dts
Info: =======================================================================
Info: get ddr information ...
Info: ddr12_size 8589934592 Byte [8192 MiB]
Info: ddr3_size 4294967296 Byte [4096 MiB]
Info: ddr4_size 4294967296 Byte [4096 MiB]
Info: ddr_size 16384 MiB
Info: =======================================================================
Info: get max memory size ...
Info: max npu size: 0x1dbf00000 [7615 MiB]
Info: max vpu size: 0xc0000000 [3072 MiB]
Info: max vpp size: 0x100000000 [4096 MiB]
Info: =======================================================================
Info: get now memory size ...
Info: now npu size: 0xf6e00000 [3950 MiB]
Info: now vpu size: 0x80000000 [2048 MiB]
Info: now vpp size: 0xc0000000 [3072 MiB]
```

### 修改内存分配设置

可参考以下命令进行内存分配设置，其中输入的三个参数是需要 NPU、VPU、VPP 配置的大小的十进制数字，单位 MiB；或者为十六进制数值，单位 Byte。

```bash
# 十进制，单位MiB
./memory_edit.sh -c -npu 2048 -vpu 2048 -vpp 2048
# 十六进制，单位Byte
./memory_edit.sh -c -npu 0x80000000 -vpu 0x80000000 -vpp 0x80000000
```

执行以上命令之一后，检查输出中是否有 Error ，以及类似于如下输出中三个部分的大小是否与所需要配置的大小相同。

```bash
Info: output configuration results ...
Info: vpu mem area(ddr3): 0x8000000 [128 MiB] 0x20000000 -> 0x27ffffff
Info: ion npu mem area(ddr1): 0x80000000 [2048 MiB] 0x24100000 -> 0xa40fffff
Info: ion vpu mem area(ddr3): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: ion vpp mem area(ddr4): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: =======================================================================
Info: start check memory size ...
Info: check npu size: 0x80000000 [2048 MiB]
Info: check vpu size: 0x80000000 [2048 MiB]
Info: check vpp size: 0x80000000 [2048 MiB]
Info: check edit size ok
Info: en_emmcfile ok
```

如检查无误，请保存当前工作，并参照以下操作​将修改后的 emmcboot.itb 文件替换启动分区中的启动映像，最后重启机器使修改生效。

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

## 开机与关机

AIBOX-1688 在连接电源时会自动开机。若已连接电源，且机器使用了关机命令，请短按电源键使机器开机。

开机进入系统后指示灯会绿色长亮。

注意：请先完成软/硬件关机后，再断开电源，以免损坏文件系统数据。

* 软件关机：在终端中运行 `sudo poweroff`。
* 按键关机：长按电源键，直至工作指示灯停止闪烁。

当风扇停止运转、工作指示灯熄灭时，说明 AIBOX-1688 已完成关机，此时可安全断开电源。

## 调试与常见问题

**系统默认的用户名和密码是什么？**

* 用户名：`linaro`
* 密码：`linaro`
* 切换超级用户： `sudo -s`

**开机异常并循环重启怎么办？**

有可能是电源电流不够，请使用电压为 12V，电流为 5A 以上的电源。

**查看内存：**

* 查看系统内存 : `free -h`
* 查看ION内存
    * NPU : `cat /sys/kernel/debug/ion/cvi_npu_heap_dump/summary`
    * VPP : `cat /sys/kernel/debug/ion/cvi_vpp_heap_dump/summary`
