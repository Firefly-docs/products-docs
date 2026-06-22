## 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu18.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

<font color=red>**非常重要：**</font>

开发前请根据 WIKI 把 SDK 更新到最新，尤其是使用 CS-R1 集群服务器用户（使用 CS-R1 用户如果代码版本太旧软件电平配置和硬件不匹配可能导致<font color=red>**硬件损坏**</font>）。

`CORE-1126-JD4/CORE-1109-JD4` 系列请下载 `rv1126_rv1109_linux_release` Linux SDK 软件包。

`CORE-1126-JD4/CORE-1109-JD4` 系列如果要进行 uvc 应用的开发，请下载 `rv1126_rv1109_linux_ai_camera_release` Linux SDK 软件包。

### SDK 开发包说明

`rv1126_rv1109_linux_release` SDK 说明：
1. `CORE-1126-JD4/CORE-1109-JD4` 系列产品可开发 IPC 固件（IPC 即网络摄像头）
2. `CAM-C1126S2U/CAM-C1109S2U` 系列产品可开发 Facial_gate 固件（Facial_gate 即人脸识别闸机）

`rv1126_rv1109_linux_ai_camera_release` SDK 说明：
1. `CORE-1126-JD4/CORE-1109-JD4` 系列产品可开发 AI_UVC 固件（UVC 即免驱摄像头）
2. `CAM-C1126S2U/CAM-C1109S2U` 系列产品可开发 AI_UVC 固件（UVC 即免驱摄像头）

完整 SDK 和 BSP SDK 说明：

`rv1126_rv1109_linux_release` 和 `rv1126_rv1109_linux_ai_camera_release` 都是完整 SDK 。

`rv1126_rv1109_linux_bsp_release` 和 `rv1126_rv1109_linux_ai_camera_bsp_release` 都是 BSP SDK 。

完整 SDK 和 BSP SDK 的区别：
1. 完整 SDK 可以编译出 buildroot 系统。文件体积比较大。里面包含 rockchip 提供的硬件加速文件目录。第一次使用建议拉取完整 SDK 。
2. BSP SDK 无法编译出 buildroot 系统。但是文件体积比较小。提供给只需要在公版固件的基础上，更新 u-boot 和 kernel 等分区固件的客户。

完整 SDK 和 BSP SDK 的相同点：

1. 都可以编译打包 Debian10 固件。

### 安装工具
获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python
```
### 初始化仓库

* 方法一（推荐国内用户使用）

**SDK 源码存放于 gitlab，国内用户可能下载完整的 SDK 仓库速度比较慢，所以我们提供了一个 SDK 基础包([Linux SDK](https://www.t-firefly.com/doc/download/98.html))，国内用户只需要在此基础包上同步 gitlab 上的代码就可以了**

注：该方法只能获取到完整 SDK 。BSP SDK 需要使用方法二获取。

* `rv1126_rv1109_linux_release` SDK 下载

下载完成后先验证一下 MD5 码：

```bash
md5sum rv1126_rv1109_linux_release_20211022.tgz
596c6bc6bb3095aea97d54c9df4cf333
```

确认无误后，就可以解压：
```bash
# 解压
mkdir -p ~/proj/rv1126_sdk
cd ~/proj/rv1126_sdk
mv /path/to/rv1126_rv1109_linux_release_20211022.tgz ./
tar xvf rv1126_rv1109_linux_release_20211022.tgz

#本压缩包内包含一个.repo目录，解压之后，在当前目录下执行以下操作
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#后续可以使用以下命令更新SDK
.repo/repo/repo sync -c --no-tags

#因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行
```

* `rv1126_rv1109_linux_ai_camera_release` SDK 下载

​	下载完成后先验证一下 MD5 码：

```bash
md5sum rv1126_rv1109_linux_ai_camera_release_20210306.tgz
26fcb17ddd82a43d396eb3eac4b7479f
```

确认无误后，就可以解压：
```bash
# 解压
mkdir -p ~/proj/rv1126_sdk_ai
cd ~/proj/rv1126_sdk_ai
mv /path/to/rv1126_rv1109_linux_ai_camera_release_20210306.tgz ./
tar xvf rv1126_rv1109_linux_ai_camera_release_20210306.tgz

#本压缩包内包含一个.repo目录，解压之后，在当前目录下执行以下操作
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

#后续可以使用以下命令更新SDK
.repo/repo/repo sync -c --no-tags

#因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行
```


* 方法二

通过 repo 拉取代码，此方法对网络要求较高，有条件可以使用。可选择获取完整 SDK 或者 BSP SDK 。

* `rv1126_rv1109_linux_release` 完整 SDK 拉取方法
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_release.xml

repo sync -c
#需反复确认代码下载是否成功
```

* `rv1126_rv1109_linux_bsp_release`  BSP SDK 拉取方法
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_bsp_release.xml

repo sync -c
#需反复确认代码下载是否成功
```



* `rv1126_rv1109_linux_ai_camera_release` 完整 SDK 拉取方法

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_release.xml

repo sync -c
#需反复确认代码下载是否成功
```



* `rv1126_rv1109_linux_ai_camera_bsp_release`  BSP SDK 拉取方法

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_bsp_release.xml

repo sync -c
#需反复确认代码下载是否成功
```



注：默认 repo 工具是使用 python2 来拉取代码。如果 Linux PC 系统比较新，可能默认的是 python3 版本。如果代码无法拉取，请手动设置一下 Linux PC 系统的 python 版本为 python2。并且使用当前目录下的 `.repo/repo/repo` 工具进行代码拉取。查看 Linux PC 系统默认的 python 版本：

```shell
$ ls -l /usr/bin/python
lrwxrwxrwx 1 root root 9 4月  16  2018 /usr/bin/python -> python2.7*
```



### 同步代码

执行如下命令同步代码：

* 进入 rv1126_rv1109_linux_release SDK 根目录

```bash
cd ~/proj/rv1126_sdk
```

* 进入 rv1126_rv1109_linux_ai_camera_release SDK 根目录

```bash
cd ~/proj/rv1126_sdk_ai
```

* 同步命令

```bash
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。
