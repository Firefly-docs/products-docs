# 编译 Android 固件

## 前提条件 [step]

编译 Android 固件对主机配置要求较高，编译前确认以下条件：

* 64 位 CPU，Ubuntu 20.04 或更高版本
* 64 GB 物理内存与交换内存
* 400GB 空闲磁盘空间
* 使用普通用户执行编译命令，不要使用 root 用户

编译环境依赖包的安装见 [编译环境搭建](android_prepare_compile_android.md)。

## 下载 SDK [step]

Firefly-R3572A 的 Android 16 SDK 通过邮件方式获取，将订单号发送至 sales@t-firefly.com 并注明需要的 SDK 名称。

SDK 源码包采用分卷压缩，包含 `firefly_rk3572_android16.0_git_20261112.7z.001` 等多个分卷文件，全部下载到同一目录后再解压。

<Callout title="注意" type="info">

本节中的源码包名与 bundle 名按实际文件名、分卷数量与 MD5 校验值以发布的 SDK 包为准。

</Callout>

下载完成后，解压前先校验 MD5：

```bash
md5sum /path/to/firefly_rk3572_android16.0_git_20261112.7z.001
...
```

将其余分卷的校验结果一并核对，确认与发布方提供的 MD5 一致后再解压：

```bash
cd ~/proj/
7z x ./firefly_rk3572_android16.0_git_20261112.7z.001 -oRK3572_Android16.0
cd ./RK3572_Android16.0
git reset --hard
```

<Callout title="注意" type="warn">

不要在共享文件夹、挂载目录或非英文路径下解压 SDK，避免产生不必要的错误。

</Callout>

## 更新 SDK [step]

SDK 通过 bundle 仓库更新。在资源下载页下载 `rk357x-android16-bundle` 压缩包，在 SDK 根目录执行：

```bash
7z x rk357x-android16-bundle.7z* -r -o. && mv rk357x-android16-bundle .bundle
.bundle/update
git rebase FETCH_HEAD
```

`.bundle/update` 将更新内容同步到 `FETCH_HEAD`，`git rebase FETCH_HEAD` 再将其合并到当前分支。

## 选择产品配置 [step]

编译前通过 `lunch` 选择 Firefly-R3572A 对应的产品配置，采用三段式名称（不省略 `bp4a`）。

在 SDK 根目录依次执行：

```bash
source build/envsetup.sh
lunch rk3572_firefly_r3572a-bp4a-userdebug
```

## 整体编译 [step]

整体编译一次性完成 U-Boot、内核、Android 的编译与镜像收集，然后单独打包统一固件，适合首次编译或完整验证。在 SDK 根目录依次执行：

```bash
./FFTools/make.sh -j8 -d rk3572-firefly-r3572a -l rk3572_firefly_r3572a-bp4a-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3572_firefly_r3572a-bp4a-userdebug
```

命令分为两段：

* `./FFTools/make.sh -j8 -d <dts> -l <lunch>`：整体编译入口，依次构建 U-Boot、内核与 Android 并收集镜像。`-d` 指定内核设备树（`rk3572-firefly-r3572a`），`-l` 指定三段式 lunch（`rk3572_firefly_r3572a-bp4a-userdebug`）。带 `-l` 而不带阶段参数时执行全部三个阶段，不是单模块入口。
* `./FFTools/mkupdate/mkupdate.sh -l <lunch>`：统一固件打包，只包装已有镜像，不编译、不清理构建目录。

## 分步编译 [step]

只需重新编译某个部分时，可进入对应目录单独编译，无需每次整体编译。以下命令均在 SDK 根目录开始执行。

### 编译内核 [step]

在 `kernel-6.12` 目录使用 `mkboot.sh` 单独编译内核并生成 `boot.img`：

```bash
cd kernel-6.12
./mkboot.sh rk3572-firefly-r3572a ../rockdev/Image-rk3572_firefly_r3572a/boot.img rk3572.config
```

`mkboot.sh` 的用法为 `./mkboot.sh <DTS 名称> <已有 boot.img 路径> <芯片 config>`：第一个参数为内核设备树 `rk3572-firefly-r3572a`，第二个参数为**已存在的 boot.img 路径**（作为输入，脚本将新编译的 kernel 与 resource 替换进去，不是输出路径），第三个参数为芯片配置 `rk3572.config`。编译完成后生成 `kernel-6.12/boot.img`。

### 编译 U-Boot [step]

在 `u-boot` 目录使用 `make.sh` 单独编译 U-Boot：

```bash
cd u-boot
./make.sh rk3572
```

### 编译 Android [step]

在 SDK 根目录编译 Android 上层：

```bash
source build/envsetup.sh
lunch rk3572_firefly_r3572a-bp4a-userdebug
make installclean
make -j8
./mkimage.sh
```

## 打包成统一固件 update.img [step]

编译完成后，可用 Firefly 官方脚本打包成统一固件：

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3572_firefly_r3572a-bp4a-userdebug
```

打包完成后将在 `rockdev/Image-rk3572_firefly_r3572a/` 目录下生成统一固件，文件名格式为 `产品名_版本_日期.img`，例如 `rk3572_firefly_r3572a_Android16_日期.img`（具体以实际生成结果为准）。

在 Windows 下打包统一固件 `update.img` 也很简单：将编译生成的文件拷贝到 AndroidTool 的 `rockdev\Image` 目录中，然后运行 `rockdev` 目录下的 `mkupdate.bat` 批处理文件，即可创建 `update.img` 并存放到 `rockdev\Image` 目录里。

## 编译产物 [step]

编译打包后，分区镜像输出到 `rockdev/` 目录，常用分区镜像如下：

| 固件 | 说明 |
| --- | --- |
| boot.img | 包含 ramdisk、kernel、dtb |
| dtbo.img | Device Tree Overlays |
| MiniLoaderAll.bin | 包含一级 loader |
| misc.img | 包含 recovery-wipe 开机标识信息 |
| parameter.txt | 包含分区信息 |
| recovery.img | 包含 recovery-ramdisk、kernel、dtb |
| super.img | 包含 odm、product、vendor、system、system_ext 分区内容 |
| uboot.img | 包含 U-Boot 固件 |
| vbmeta.img | 包含 AVB 校验信息 |
| update.img | 包含以上需要烧写的镜像，供工具直接烧写整个固件包 |

Firefly-R3572A 的镜像输出目录为 `rockdev/Image-rk3572_firefly_r3572a/`，预期包含 `boot.img`、`super.img`、`update.img` 等文件。

可用 `ls rockdev/` 确认产物。烧录方法见 [使用 USB 线缆升级固件](../../getting_started/firmware_upgrade/upgrade_firmware.md)。

## OTA 编译 [step]

### 完整 OTA 包 [step]

在整体编译命令上追加 `-o` 参数，即可进行 OTA 全局编译并生成完整 OTA 包：

```bash
./FFTools/make.sh -j8 -d rk3572-firefly-r3572a -l rk3572_firefly_r3572a-bp4a-userdebug -o
```

生成的完整 OTA 包路径为：

```bash
out/target/product/rk3572-firefly-r3572a/rk3572-firefly-r3572a-ota-eng.xxx.zip
```

基础素材包（用于生成差分包）路径为：

```bash
out/target/product/rk3572-firefly-r3572a/obj/PACKAGING/target_files_intermediates/rk3572-firefly-r3572a-target_files-eng.xxx.zip
```

拷出基础素材包并重命名，供后续制作差分包：

```bash
mkdir -p ~/ota
cp out/target/product/rk3572-firefly-r3572a/obj/PACKAGING/target_files_intermediates/rk3572-firefly-r3572a-target_files-eng.xxx.zip ~/ota/v1.zip
```

### OTA 差分包 [step]

制作差分包需要两个版本的基础素材包。先按上述方法得到旧版 `v1.zip`，修改内核 DTS 或 Android 代码后，再次执行带 `-o` 的编译生成新版素材包，并拷出为 `v2.zip`：

```bash
./FFTools/make.sh -j8 -d rk3572-firefly-r3572a -l rk3572_firefly_r3572a-bp4a-userdebug -o
cp out/target/product/rk3572-firefly-r3572a/obj/PACKAGING/target_files_intermediates/rk3572-firefly-r3572a-target_files-eng.xxx.zip ~/ota/v2.zip
```

然后生成差分包：

```bash
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ \
    -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```

将生成的 `update.zip` 放到设备 `/sdcard/` 目录后重启，系统会识别并提示完成 OTA 升级（完整包与差分包升级方式相同，均把 `update.zip` 放入 `/sdcard/` 重启）。

<Callout title="注意" type="info">

以上 OTA 命令与路径以 Firefly-R3572A 的 Android 16 SDK 实际生成为准；OTA 服务器搭建请参考 SDK 提供的 OTA 文档。

</Callout>

## 常见问题 [step]

### Q1：更新 SDK 时 git clone 远程 bundle 仓库失败

A1：缓冲区不足时扩大缓冲区后重试：

```bash
git config --global https.postBuffer 1048576000
```