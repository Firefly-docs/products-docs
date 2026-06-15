
### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[rk356x_linux5.10_release_20241220_v1.4.0c]()

下载 SDK 基础包并且按照 README 文档操作：

```bash
└── rk356x_linux5.10_release_xxx
    ├── linux_sdk_tar
    │   ├── rk356x_linux5.10_release_xxx.sdk.split00
    │   ├── rk356x_linux5.10_release_xxx.sdk.split01
    │   ├── rk356x_linux5.10_release_xxx.sdk.split02
    │   ├── rk356x_linux5.10_release_xxx.sdk.split03
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
# 创建 SDK 目录
mkdir ~/proj/rk356x_sdk-linux5.10
cd ~/proj/rk356x_sdk-linux5.10

## 完整 SDK
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk356x_linux5.10_release.xml

## BSP （ 只包含基础仓库和编译工具只适合编译 ubuntu、debian 固件 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk356x_linux5.10_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_sdk-linux5.10

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
├── device															# 编译相关配置文件
├── docs															# 文档
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts														# 交叉编译工具链
├── rkbin
├── tools															# 工具目录
└── u-boot															# U-Boot
```
