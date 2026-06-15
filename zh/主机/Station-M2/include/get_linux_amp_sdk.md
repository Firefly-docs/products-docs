
### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[rk356x_amp_release_20240607_v0.0.1a]()

下载完成后，提取 SDK：

```bash
# 给脚本添加执行权限
cd /path/to/rk356x_amp_release_20240607_v0.0.1a
chmod +x ./sdk_tools.sh

# 创建 SDK 目录
mkdir -p ~/proj/rk356x_amp_sdk

# 校验并解压
./sdk_tools.sh --unpack -C ~/proj/rk356x_amp_sdk

# 还原工作目录
./sdk_tools.sh --sync -C ~/proj/rk356x_amp_sdk
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
# 创建 SDK 目录
mkdir ~/proj/rk356x_amp_sdk
cd ~/proj/rk356x_amp_sdk

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、hal、 rt-thread 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_amp_sdk

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
├── buildroot   # Buildroot 根文件系统编译目录
├── build.sh    # 编译脚本
├── device      # 编译相关配置文件
├── docs        # SDK 相关文档
├── envsetup.sh
├── external
├── hal
│ ├── doc       # HAL 相关文档
│ ├── html      # HAL doxygen 生成驱动文档
│ ├── lib       # HAL 驱动代码
│ ├── project   # HAL 工程目录
│ ├── test      # HAL 测试代码
│ └── tools     # HAL 相关工具
├── kernel      # Kernel相关代码
├── rt-thread   # RT-Thread相关代码
├── prebuilts   # SDK 编译工具链(包括编译U-Boot、Kernel、HAL编译工具链)
├── rkbin       # SDK 相关Binary文件（包括Loader、Trust等打包用基础bin文件）
├── tools
│ ├── linux     # Linux 平台烧录工具
│ ├── mac       # Mac 平台烧录工具
│ └── windows   # Windows 平台烧录工具及驱动
└── u-boot      # U-boot相关代码
```
