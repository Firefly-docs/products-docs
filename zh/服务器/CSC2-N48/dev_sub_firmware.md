# 定制子节点固件



## 固件预处理

`firmware-kits` 用于在正式固件升级前，对 BM1684 和 RK3588 固件包进行预处理。它可以按指定流程解包原始固件、调整 ROOTFS/DATA/CUSTOM 等分区大小、挂载 rootfs 供人工修改系统内容，并在修改完成后重新打包生成可用于交付或升级的固件包。

该工具适用于需要在升级前定制固件内容的场景，例如扩容分区、预置文件、调整 rootfs 配置、生成 TFTP 或 SD 卡升级包。普通固件升级仍应优先使用本页面前文的升级管理功能；只有在固件需要先被修改或重新打包时，才使用 `firmware-kits`。

下面说明如何使用 `firmware-kits` 对 BM1684 和 RK3588 固件进行解包、修改和重新打包。请严格按步骤操作。

所有命令都在 `firmware-kits` 目录下执行。

### 1. 准备工作

#### 1.1 操作系统要求

推荐使用 Ubuntu 20.04/22.04 x86_64 主机。

#### 1.2 安装依赖

```bash
sudo apt-get update
sudo apt-get install -y u-boot-tools qemu-user-static p7zip-full unzip e2fsprogs
```

#### 1.3 检查工具包目录

进入工具包目录：

```bash
cd firmware-kits
```

确认目录中至少有这些内容：

```text
firmware-kits
flow/
scripts/
tools/
bin/
```

确认主程序可以执行：

```bash
./firmware-kits --help
```

固件处理会执行挂载、chroot、文件系统调整和重新打包，所以正式操作时需要使用 `sudo`。

### 2. 基本命令

从头开始处理固件：

```bash
sudo ./firmware-kits run -l <流程文件> -f <固件文件>
```

流程暂停后继续：

```bash
sudo ./firmware-kits resume
```

参数说明：

| 参数 | 说明 |
| ---- | ---- |
| `run` | 从头开始执行一个固件处理流程 |
| `resume` | 从上一次暂停的位置继续执行 |
| `-l` | 指定流程文件，例如 `flow/bm1684_tftp_data.yaml` |
| `-f` | 指定原始固件文件 |
| `-o` | 高级参数，追加传给处理脚本；没有特别说明时不要使用 |

工具执行到需要人工处理的位置时会自动暂停，并提示下一步要执行什么命令。人工操作完成后，执行 `sudo ./firmware-kits resume` 继续。

### 3. 重要注意事项

1. 不要在流程暂停时删除 `out/`、`.firmware-kits_state.json` 或 `tools/.env`。
2. 不要关闭已经挂载 rootfs 后的主机，除非已经确认卸载完成。
3. 每次只处理一个固件流程，不要同时开多个终端跑多个 `firmware-kits run/resume`。
4. 输出目录是 `out/`。确认不需要产物后，可以用 `sudo rm -rf out/` 清理空间。
5. 如果流程提示存在中断记录，优先执行 `sudo ./firmware-kits resume`，不要直接重新 `run`。

### 4. BM1684 固件操作

#### 4.1 输入文件

BM1684 使用 zip 固件包，例如：

```text
Public-All-1684-Ubuntu2004-Sdk2309LTS_SP5-Build20260522.zip
```

把固件文件放到 `firmware-kits` 目录下后执行后续命令。

#### 4.2 选择输出类型和 flow

BM1684 有多个 flow。不同 flow 的区别主要是输出类型、是否处理 BOOT/DATA 分区、是否处理 CUSTOM 分区。

| flow 文件 | 适用场景 | 输出产物 | 暂停点 | 关键参数 |
| ---- | ---- | ---- | ---- | ---- |
| `flow/bm1684.yaml` | 基础 SD 卡升级包；只处理 ROOTFS | `out/bm1684/ubuntu20/update/sdcard-YYYY-MM-DD.zip` | ROOTFS 分区大小、rootfs 内容修改 | 打包参数 `sdcard` |
| `flow/bm1684_tftp.yaml` | 基础 TFTP 升级包；只处理 ROOTFS | `out/bm1684/ubuntu20/update/tftp-YYYY-MM-DD.zip` | ROOTFS 分区大小、rootfs 内容修改 | 打包参数 `tftp` |
| `flow/bm1684_sdcard_data.yaml` | SD 卡升级包；处理 ROOTFS、BOOT、DATA | `out/bm1684/ubuntu20/update/sdcard-YYYY-MM-DD.zip` | ROOTFS 分区大小、DATA 分区大小、rootfs 内容修改 | 打包参数 `sdcard`；BOOT 自动处理 |
| `flow/bm1684_tftp_data.yaml` | TFTP 升级包；处理 ROOTFS、BOOT、DATA | `out/bm1684/ubuntu20/update/tftp-YYYY-MM-DD.zip` | ROOTFS 分区大小、DATA 分区大小、rootfs 内容修改 | 打包参数 `tftp`；BOOT 自动处理 |
| `flow/bm1684_custom_overlay.yaml` | TFTP 升级包；增加或扩容 CUSTOM，并将 CUSTOM/DATA bind 到 rootfs | `out/bm1684/ubuntu20/update/tftp-YYYY-MM-DD.zip` | ROOTFS 分区大小、CUSTOM 分区大小、DATA 分区大小、rootfs 内容修改 | CUSTOM bind 参数中默认大小为 `20`；打包参数 `tftp` |
| `flow/bm1684_custom_ro.yaml` | TFTP 升级包；CUSTOM 按只读方案处理，同时处理 BOOT/DATA | `out/bm1684/ubuntu20/update/tftp-YYYY-MM-DD.zip` | ROOTFS 分区大小、DATA 分区大小、rootfs 内容修改 | 打包参数 `tftp custom_ro`；CUSTOM/BOOT 自动处理 |

说明：

1. 输入固件都是 BM1684 zip 包，例如 `Public-All-1684-Ubuntu2004-Sdk2309LTS_SP5-Build20260522.zip`。
2. `ROOTFS 分区大小` 暂停点可以查看或扩容 ROOTFS。
3. `DATA 分区大小` 暂停点可以查看或扩容 DATA。只有带 `_data` 或 custom 的 flow 才会出现。
4. `CUSTOM 分区大小` 暂停点只在 `bm1684_custom_overlay.yaml` 中出现。
5. 如果不需要修改某个暂停点的内容，直接执行 `sudo ./firmware-kits resume` 即可。

常用命令示例：

生成 TFTP 升级包，处理 ROOTFS/BOOT/DATA：

```bash
sudo ./firmware-kits run -l flow/bm1684_tftp_data.yaml -f ./Public-All-1684-Ubuntu2004-Sdk2309LTS_SP5-Build20260522.zip
```

生成 SD 卡升级包，处理 ROOTFS/BOOT/DATA：

```bash
sudo ./firmware-kits run -l flow/bm1684_sdcard_data.yaml -f ./Public-All-1684-Ubuntu2004-Sdk2309LTS_SP5-Build20260522.zip
```

只处理 ROOTFS 并生成 TFTP 升级包：

```bash
sudo ./firmware-kits run -l flow/bm1684_tftp.yaml -f ./Public-All-1684-Ubuntu2004-Sdk2309LTS_SP5-Build20260522.zip
```

只处理 ROOTFS 并生成 SD 卡升级包：

```bash
sudo ./firmware-kits run -l flow/bm1684.yaml -f ./Public-All-1684-Ubuntu2004-Sdk2309LTS_SP5-Build20260522.zip
```

如果没有特别要求，按交付人员指定的 flow 执行。

#### 4.3 第一次暂停：修改 ROOTFS 分区大小

程序会显示类似信息：

```text
[Step 3/13] 修改根文件系统大小
固件处理已暂停，客户完成上述手动操作后，可输入 `sudo ./firmware-kits resume` 进行继续处理
```

查看 ROOTFS 当前大小：

```bash
sudo ./tools/chroot-p-bm1684.sh -p ROOTFS -r
```

如果需要扩容，例如扩容到 4000 MB：

```bash
sudo ./tools/chroot-p-bm1684.sh -p ROOTFS -w 4000
```

如果不需要修改大小，可以直接继续：

```bash
sudo ./firmware-kits resume
```

如果已经修改完大小，也执行同一个继续命令：

```bash
sudo ./firmware-kits resume
```

#### 4.4 第二次暂停：修改 DATA 分区大小

程序会显示类似信息：

```text
[Step 7/13] 修改 DATA 分区的大小
固件处理已暂停，客户完成上述手动操作后，可输入 `sudo ./firmware-kits resume` 进行继续处理
```

查看 DATA 当前大小：

```bash
sudo ./tools/chroot-p-bm1684.sh -p DATA -r
```

如果需要扩容，例如扩容到 6000 MB：

```bash
sudo ./tools/chroot-p-bm1684.sh -p DATA -w 6000
```

如果不需要修改大小，可以直接继续：

```bash
sudo ./firmware-kits resume
```

#### 4.5 第三次暂停：修改 rootfs 内容

程序会显示类似信息：

```text
[Step 9/13] 修改根文件系统内容
固件处理已暂停，客户完成上述手动操作后，可输入 `sudo ./firmware-kits resume` 进行继续处理
```

在 rootfs 内执行命令：

```bash
sudo ./tools/chroot-e.sh 'apt-get update'
```

安装软件包示例：

```bash
sudo ./tools/chroot-e.sh 'apt-get install -y figlet'
```

复制本地文件到 rootfs，例如把当前目录的 `a.txt` 复制到目标系统的 `/home/`：

```bash
sudo ./tools/chroot-c.sh -s ./a.txt -d /home/
```

说明：

| 操作 | 命令 |
| ---- | ---- |
| 执行一条 rootfs 内命令 | `sudo ./tools/chroot-e.sh 'COMMAND'` |
| 复制文件或目录到 rootfs | `sudo ./tools/chroot-c.sh -s <本地路径> -d <rootfs内目标路径>` |

修改完成后继续：

```bash
sudo ./firmware-kits resume
```

之后程序会自动卸载 DATA、BOOT、ROOTFS，并重新打包固件。

#### 4.6 BM1684 输出文件

TFTP flow 输出：

```text
out/bm1684/ubuntu20/update/tftp-YYYY-MM-DD.zip
```

SD 卡 flow 输出：

```text
out/bm1684/ubuntu20/update/sdcard-YYYY-MM-DD.zip
```

实际验证示例：

```text
out/bm1684/ubuntu20/update/tftp-2026-07-28.zip
```

输出文件可能属于 root 用户。如果复制或删除时权限不足，请使用 `sudo`。

### 5. RK3588 固件操作

#### 5.1 输入文件

RK3588 支持直接输入 `.7z`、`.zip` 或 `.img`。例如：

```text
CS-B1-N10-SUB-3588JD4_Debian12-Xfce-r1275_debug_260710.7z
```

工具会自动从 `.7z/.zip` 中解出 `.img` 后继续处理。

#### 5.2 开始处理

```bash
sudo ./firmware-kits run -l flow/rk3588.yaml -f ./CS-B1-N10-SUB-3588JD4_Debian12-Xfce-r1275_debug_260710.7z
```

#### 5.3 第一次暂停：修改 ROOTFS 分区大小

程序会显示类似信息：

```text
[Step 3/7] 修改根文件系统大小
固件处理已暂停，客户完成上述手动操作后，可输入 `sudo ./firmware-kits resume` 进行继续处理
```

查看 rootfs 当前大小：

```bash
sudo ./tools/chroot-p-rk3588.sh -p rootfs -r
```

如果需要扩容，例如扩容到 4000 MB：

```bash
sudo ./tools/chroot-p-rk3588.sh -p rootfs -w 4000
```

如果不需要修改大小，可以直接继续：

```bash
sudo ./firmware-kits resume
```

#### 5.4 第二次暂停：修改 rootfs 内容

程序会显示类似信息：

```text
[Step 5/7] 修改根文件系统内容
固件处理已暂停，客户完成上述手动操作后，可输入 `sudo ./firmware-kits resume` 进行继续处理
```

在 rootfs 内执行命令：

```bash
sudo ./tools/chroot-e.sh 'apt-get update'
```

复制文件到 rootfs：

```bash
sudo ./tools/chroot-c.sh -s ./a.txt -d /home/
```

修改完成后继续：

```bash
sudo ./firmware-kits resume
```

之后程序会自动卸载 rootfs，并重新打包 RK3588 固件。

#### 5.5 RK3588 输出文件

输出文件位置：

```text
out/rk3588/ubuntu20/update/rockdev/rk3588-YYYY-MM-DD.img
```

实际验证示例：

```text
out/rk3588/ubuntu20/update/rockdev/rk3588-2026-07-28.img
```

输出文件可能属于 root 用户。如果复制或删除时权限不足，请使用 `sudo`。

### 6. 中断恢复

如果执行 `run` 时看到类似提示：

```text
发现中断记录，请使用 `sudo ./firmware-kits resume` 继续执行
```

说明上一次流程没有完成。请执行：

```bash
sudo ./firmware-kits resume
```

如果确认上一次流程不要了，再清理状态和输出：

```bash
sudo rm -f .firmware-kits_state.json tools/.env
sudo rm -rf out/
```

清理后可以重新执行 `run`。

### 7. 执行记录

随包提供两份实际执行记录，可下载后查看完整操作过程：

- [下载 BM1684 实际执行记录](/docs-assets/server-docs/aBMC_img/v3.0/firmware-kits/bm1684-run-2026-07-28.zip)
- [下载 RK3588 实际执行记录](/docs-assets/server-docs/aBMC_img/v3.0/firmware-kits/rk3588-run-2026-07-28.zip)

日志中只保留客户关心的固件处理命令、终端输出、暂停点操作、rootfs 内执行命令演示、拷贝文件演示、最终产物路径和挂载检查结果。

### 8. firmware-kits 常见问题

#### 8.1 提示权限不足

固件处理涉及挂载、chroot 和 root-owned 文件。请确认命令使用了 `sudo`。

如果删除输出目录失败，执行：

```bash
sudo rm -rf out/
```

#### 8.2 不确定当前该执行什么

如果流程暂停，终端通常会提示下一步命令。一般只需要：

1. 按提示查看或修改分区大小，或修改 rootfs 内容。
2. 完成后执行 `sudo ./firmware-kits resume`。

#### 8.3 不需要修改分区大小怎么办

在分区大小暂停点，如果不需要修改，直接执行：

```bash
sudo ./firmware-kits resume
```

#### 8.4 不需要修改 rootfs 内容怎么办

在 rootfs 内容修改暂停点，如果不需要修改，直接执行：

```bash
sudo ./firmware-kits resume
```

#### 8.5 apt 命令出现 locale 或 /dev/pts 警告

部分 rootfs 环境可能显示 locale 或 `/dev/pts` 相关警告。只要命令最终执行成功，一般不影响固件制作。

#### 8.6 如何确认没有残留挂载

正常流程完成后通常不会残留挂载。需要检查时执行：

```bash
mount | grep firmware-kits/out
```

没有输出表示没有残留挂载。

如果有输出，请联系交付人员处理，不要随意删除正在挂载的目录。