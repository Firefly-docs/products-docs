## 编译环境搭建

本章介绍 Linux SDK的编译流程

<font color=red>**注意：（1）推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。**</font>

<font color=red>**（2）使用普通用户进行编译，不要使用 root 用户权限进行编译。**</font>

### 安装依赖包

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
```

### 下载 Firefly_Linux_SDK 分卷压缩包

* 方法一（推荐）

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[Firefly_Linux_SDK源码包](https://www.t-firefly.com/doc/download/102.html#other_300)

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3308_linux_release_v1.5.0a_20221212_split_dir/*firefly_split*
a97f218aed44be1b9a423a2868e09335  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file0
3f3936a72635b75613b89a4bf8d9de39  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file1
794ea504543a797d972c2112dfef2f3f  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file2
0de1858d73e2841790006e487ff70a19  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file3
```

确认无误后，就可以解压：

<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

```bash
# 解压
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3308_linux_release_v1.5.0a_20221212_split_dir/*firefly_split* | tar -xzv

# 导出数据
.repo/repo/repo sync -l
```

* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用

可选择获取完整 SDK 或者 BSP：

```bash
mkdir ~/proj/rk3308_linux_release_v1.5.0a_20221212/
cd ~/proj/rk3308_linux_release_v1.5.0a_20221212/

## 完整 SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3308_linux_release.xml

## BSP （ 只包含基础仓库和编译工具 ）
## BSP 包括 device/rockchip 、docs 、 kernel 、 u-boot 、 rkbin 、 tools 和交叉编译链
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3308_linux_bsp_release.xml
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk3308_linux_release_v1.5.0a_20221212/

# 同步
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。