### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[Firefly_Linux_SDK源码包]()

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split*
409b81a9ed3bb9a7d6af91223836cad5  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file00
75cef82f2bf91052a7d3c6f0b8405a89  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file01
6f20f62e9652f8f999692587a2ac4b79  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file02
113acbbcd18d3abe0552ef296e983a3f  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file03
624c88a4da2eaa4a48f380783b126d00  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file04
1cf861afb0b36c9ebcf26a7d6effb260  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file05
3009e46fc14481e77fe7ec143e217de4  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file06
5e1cc90b99e34f20b75fb506d3e9bcd7  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file07
1a512fa7c9e2fd1a0781f8d40e228402  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file08
```

确认无误后，就可以解压：
```bash
# 解压
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split* | tar -xzv

# 导出数据
.repo/repo/repo sync -l
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
mkdir ~/proj/rk356x_linux_release_v1.3.0b_20221213/
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。

### 目录结构

```bash
.
├── app
├── buildroot														# Buildroot 根文件系统编译目录
├── build.sh -> device/rockchip/common/build.sh					# 编译脚本
├── debian															# Debian 根文件系统编译目录
├── device															# 编译相关配置文件
├── docs															# 文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh	# 链接脚本
├── prebuilts														# 交叉编译工具链
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh				# 烧写脚本
├── tools															# 工具目录
└── u-boot															# U-Boot
```