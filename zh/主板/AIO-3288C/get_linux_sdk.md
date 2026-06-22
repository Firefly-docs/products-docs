### 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>**不要在虚拟机共享文件夹以及非英文目录存放、解压SDK，避免产生不必要的错误**</font>

获取 SDK 需要先安装：

```
sudo apt update
sudo apt install -y repo git python
```

* 方法一（推荐）

**Firefly_Linux_SDK 源码包比较大，可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[下载链接](http://www.t-firefly.com/doc/download/51.html#other_207)

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split*
f3b9309c574e04491a9e9f3e7a5c5540  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file0
fb0cd76f518405441bbc2ad2334ee6b9  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file1
cb26fddb36d49ff6b79f2fc934a731c1  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file2
43aa8447d4ef303ce41274c6ea7b00a2  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file3
c6c1ed77033af3d26be24974dda1b2aa  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file4
```

确认无误后，就可以解压：

```bash
# 解压
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split* | tar -xzv

# 导出数据
.repo/repo/repo sync -l
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
mkdir ~/proj/rk3288_linux_release_v2.5.0a_20230510/
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用如下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。