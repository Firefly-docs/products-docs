# 硬件功能使用
## 调试串口

AIBOX-1684 采用板载 Type C 调试串口，使用 Type C 转 USB 数据线连接机器的 Type C 口和 PC 的 USB 口即可直连调试，登录账号、密码均为 `linaro`。

串口参数为：

* 波特率：115200
* 数据位：8
* 停止位：1
* 奇偶校验：无
* 流控：无

### Windows 上使用串口调试

Windows 上一般用 putty 或 SecureCRT 软件，这里推荐使用 MobaXterm 免费版本，这是一款功能强大的终端软件，其它串口软件的使用方法与之类似。

到这里[下载 MobaXterm](https://mobaxterm.mobatek.net/)：

1. 选择 `session` 为 `Serial`。
2. 将 `Serial port` 修改为在设备管理器中找到的 COM 端口。
3. 设置 `Speed (bsp)` 为 `115200`。
4. 点击 `OK` 按钮。

<center>

<img alt="" src="../../../bm1684_img/debug_set_MobaXterm1.PNG" width="800">
</center>
<center>

<img alt="" src="../../../bm1684_img/debug_set_MobaXterm2.PNG" width="800">
</center>

### Linux 上使用串口调试

在 Linux 上可以有多种选择：minicom、picocom、kermit。篇幅关系，以下介绍 minicom 的使用。

安装 minicom：

```
sudo apt-get install minicom
```

连接好串口线后，看一下串口设备文件是什么，下面示例是 `/dev/ttyUSB0`：

```
$ ls /dev/ttyUSB*
/dev/ttyUSB0
```

运行：

```
$ sudo minicom
Welcome to minicom 2.7
OPTIONS: I18n
Compiled on Jan  1 2014, 17:13:19.
Port /dev/ttyUSB0, 15:57:00
Press CTRL-A Z for help on special keys
```

以上提示 `CTRL-A Z` 是转义键，按 `Ctrl-a` 然后再按 `z` 就可以调出菜单：

```
   +-------------------------------------------------------------------+
                         Minicom Command Summary                      |
  |                                                                    |
  |              Commands can be called by CTRL-A <key>                |
  |                                                                    |
  |               Main Functions                  Other Functions      |
  |                                                                    |
  | Dialing directory..D  run script (Go)....G | Clear Screen.......C  |
  | Send files.........S  Receive files......R | cOnfigure Minicom..O  |
  | comm Parameters....P  Add linefeed.......A | Suspend minicom....J  |
  | Capture on/off.....L  Hangup.............H | eXit and reset.....X  |
  | send break.........F  initialize Modem...M | Quit with no reset.Q  |
  | Terminal settings..T  run Kermit.........K | Cursor key mode....I  |
  | lineWrap on/off....W  local Echo on/off..E | Help screen........Z  |
  | Paste file.........Y  Timestamp toggle...N | scroll Back........B  |
  | Add Carriage Ret...U                                               |
  |                                                                    |
  |             Select function or press Enter for none.               |
  +--------------------------------------------------------------------+
```

根据提示按 `O` 进入设置界面：

```
           +-----[configuration]------+
           | Filenames and paths      |
           | File transfer protocols  |
           | Serial port setup        |
           | Modem and dialing        |
           | Screen and keyboard      |
           | Save setup as dfl        |
           | Save setup as..          |
           | Exit                     |
           +--------------------------+
```

把光标移动到 `Serial port setup`，按 enter 进入串口设置界面，再输入前面提示的字母，选择对应的选项，设置成如下：

```
   +-----------------------------------------------------------------------+
   | A -    Serial Device      : /dev/ttyUSB0                              |
   | B - Lockfile Location     : /var/lock                                 |
   | C -   Callin Program      :                                           |
   | D -  Callout Program      :                                           |
   | E -    Bps/Par/Bits       : 115200 8N1                                |
   | F - Hardware Flow Control : No                                        |
   | G - Software Flow Control : No                                        |
   |                                                                       |
   |    Change which setting?                                              |
   +-----------------------------------------------------------------------+
```

**注意：** `Hardware Flow Control` 和 `Software Flow Control` 都要设成 No，否则可能导致无法输入。

设置完成后回到上一菜单，选择 `Save setup as dfl` 即可保存为默认配置，以后将默认使用该配置。

## 登录与网络配置

### 登录

AIBOX-1684 支持调试串口登录和以太网 SSH 登录两种方式，登录账号与密码均为 `linaro`。

* 调试串口登录：使用方法见上文「调试串口」。
* 以太网 SSH 登录：AIBOX-1684 提供 2 个千兆以太网接口，默认出厂时网口 0（靠近 USB 口）设置了动态 IP，网口 1（靠近 12V 电源接口）设置了静态 IP `192.168.150.1`，子网掩码 `255.255.255.0`。初次登录建议选择网口 1，先将 PC 设置成 `192.168.150.2/24` 同网段 IP，在网口灯闪烁正常后，打开终端使用 `ssh` 登录，端口号为 `22`：

  ```shell
  ssh linaro@192.168.150.1
  ```

若登录失败，可以尝试用 PC `ping` AIBOX-1684 网口 1 的 IP 地址，注意的是 PC 机要添加同网段 IP，原因是 PC 机的 IP 地址不一定处于同一网段。

以下是 PC 添加同网段 IP 的方法（使用管理员模式）：

- windows 10

  ```
  netsh int ipv4 add address "以太网" 192.168.150.101 255.255.255.0
  ```
  其中 `"以太网"` 指的是网络接口，`192.168.150.101` 是 IP 地址，`255.255.255.0` 则为子网掩码。

- linux

  ```shell
  ifconfig enp4s0:1 192.168.150.89
  ```
  其中 `enp4s0` 是 PC 机的网卡驱动名称，具体需要执行 `ifconfig` 命令查看。

### 网络 IP 配置

在默认情况中，AIBOX-1684 的网口 0 设置了动态 IP，网口 1 设置了静态 IP。这些配置都是通过系统中的 `/etc/netplan/01-netcfg.yaml` 文件实现的。

```shell
linaro@aibox-1684x:~$ cat /etc/netplan/01-netcfg.yaml
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

用户可通过修改 `/etc/netplan/01-netcfg.yaml` 文件达到修改网络配置的目的，举例如：

- 配置网口 0 为静态 IP：参考原 `eth1` 设置修改 `eth0` 节点，将 `dhcp4` 参数修改为 `no`，并添加 IP 地址进 `addresses` 参数中。

- 配置网口 1 为动态 IP：参考原 `eth0` 设置修改 `eth1` 节点，将 `dhcp4` 参数修改为 `yes`，并去除 `addresses` 参数中的 IP 地址。

在完成 `/etc/netplan/01-netcfg.yaml` 文件的修改后，用户可以通过执行 `sudo netplan apply` 命令使设置立即生效。

## USB 接口

AIBOX-1684 提供 2 个 USB 3.0 接口。插入 U 盘后，存储设备会被识别为 `/dev/sdb1` 之类的节点，系统不支持自动挂载，需要手工进行 `mount` 挂载操作。

插入 U 盘，使用 `dmesg` 可知道 U 盘对应的 `sd` 设备：

```shell
[15460.953423] [5]  sdb: sdb1
```

挂载 U 盘：

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

AIBOX-1684 提供 1 个 TF 卡座，可用于挂载 TF 卡扩展存储，也可以通过 TF 卡升级系统固件。

TF 卡挂载与 U 盘是类似的，使用 `dmesg` 可知道 TF 卡对应的 `mmcblk` 设备：

```shell
[16220.776440] [4]  mmcblk1: p1
```

挂载 TF 卡：

```shell
# 创建挂载目录
mkdir media
# 挂载
sudo mount /dev/mmcblk1p1 media
# 查看 TF 卡内的文件
ls media
```

TF 卡升级固件的方法请参阅[系统固件升级](fw-upgrade-by-sdcard.md)。

## 风扇

AIBOX-1684 风扇工作分为 5 个等级：

| 主控当前温度 | 风扇工作等级 |
|:----------:|:----------:|
| ≥ 33℃ | 1 |
| ≥ 55℃ | 2 |
| ≥ 60℃ | 3 |
| ≥ 65℃ | 4 |
| ≥ 70℃ | 5 |

风扇工作等级越高，转速越快，如果主控当前温度低于 33℃，风扇默认是关闭状态。

用户可通过执行以下命令查看当前主控的工作温度：

```shell
cat /sys/class/thermal/thermal_zone1/temp
```

例如以下是 45.5 ℃：

```shell
linaro@bm1684:~$ cat /sys/class/thermal/thermal_zone1/temp
45500
```

另外，用户也可以自己手动打开风扇（写入 0-5， 0 是关闭， 5 是最大等级）：

```
sudo -i
echo 4 > /sys/class/thermal/cooling_device0/cur_state
```

## 内存

AIBOX-1684 的内存有一部分被分配于 NPU、VPP 和 VPU 设备，因此使用 `free -h` 等指令获取的内存数据与 AIBOX-1684 的实际内存不一致。

如需查看详细的内存分配设置，或者默认内存分配与实际需求冲突、需调整内存分配的，可参照以下步骤进行相应操作。

### 安装内存分配工具

从[下载中心]中获取内存分配工具压缩包，并将其传输到 AIBOX-1684 上，执行以下命令解压压缩包：

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

### 查看内存分配设置

在完成压缩包解压后，执行以下命令查看内存分配设置：

```bash
cd memory_edit
./memory_edit.sh -p
```

内存分配设置结果示例如下，其中 `Info: get max memory size ...` 显示的结果是每个设备可配置的最大内存，`Info: get now memory size ...` 显示的结果是每个设备目前所分配的内存。

```bash
linaro@aibox-1684x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1684x_se7_v1_mini.dts
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

执行以上命令之一后，检查输出中是否有 Error，以及类似于如下输出中三个部分的大小是否与所需要配置的大小相同。

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

如检查无误，请保存当前工作，并参照以下操作将修改后的 `emmcboot.itb` 文件替换启动分区中的启动映像，最后重启机器使修改生效。

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

## 常见问题

### 系统默认的用户名和密码是什么？

* 用户名：`linaro`
* 密码：`linaro`
* 切换超级用户： `sudo -s`

### 开机异常并循环重启怎么办？

有可能是电源电流不够，请使用电压为 12V，电流为 5A 以上的电源。

[下载中心]: https://community.t-firefly.com/doc/download/280