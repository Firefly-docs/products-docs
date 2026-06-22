
### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。请联系商务 sales@t-firefly.com 获取下载地址。**

下载完成后，提取 SDK：

```bash
# 给脚本添加执行权限
cd /path/to/rk3562_linux_release
chmod +x ./sdk_tools.sh

# 创建 SDK 目录
mkdir -p ~/proj/rk3562_sdk

# 校验并解压
./sdk_tools.sh --unpack -C ~/proj/rk3562_sdk

# 还原工作目录
./sdk_tools.sh --sync -C ~/proj/rk3562_sdk
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
# 创建 SDK 目录
mkdir ~/proj/rk3562_sdk
cd ~/proj/rk3562_sdk

# 请联系商务 sales@t-firefly.com 获取 repo 链接。
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk3562_sdk

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
├── buildroot                                               # buildroot 仓库
├── build.sh -> device/rockchip/common/scripts/build.sh     # 编译脚本
├── device/rockchip                                         # 管理编译与配置的仓库
├── docs                                                    # 文档
├── external                                                # 各种组件
├── kernel                                                  # 内核仓库
├── Makefile -> device/rockchip/common/Makefile
├── output                                                  # 编译输出目录
├── prebuilt_rootfs                                         # 预编译的 rootfs 存放目录
├── prebuilts/gcc/linux-x86/                                # 交叉编译工具链
├── README.md -> device/rockchip/common/README.md
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rockdev -> output/firmware                              # 存放编译出的各种分区镜像
├── tools                                                   # 各种工具，如烧录工具等
└── u-boot                                                  # uboot 仓库
```
