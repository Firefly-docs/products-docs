## 获取 SDK

首先准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

### 安装工具
获取 SDK 需要先安装：
```
sudo apt update
sudo apt install -y repo git python p7zip-full
```
### 初始化仓库

**我们提供了一个 SDK 基础包。请联系业务 sales@t-firefly.com 获取。**

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3506_linux_release_20250117_v1.0.0a/*
a5df458069569e9288b1d96097b43397  rk3506_linux_release_20250117_v1.0.0a.7z.001
7c1a1b050ae4f0daf64fe7488a7f4b02  rk3506_linux_release_20250117_v1.0.0a.7z.002
c7998713e7ef3512635ad2f79730d888  rk3506_linux_release_20250117_v1.0.0a.7z.003
```

确认无误后，就可以解压：
```bash
# 解压
mkdir -p ~/proj/rk3506_sdk
cd ~/proj/rk3506_sdk
7z x /path/to/rk3506_linux_release_20250117_v1.0.0a/rk3506_linux_release_20250117_v1.0.0a.7z.001
```