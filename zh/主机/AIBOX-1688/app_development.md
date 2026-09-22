# 应用开发

## 编译 BSP

## 介绍

BSP 是介于底层硬件与上层软件之间的底层软件开发包，它主要功能为屏蔽硬件，提供操作系统驱动及硬件驱动。BSP 是用于构建系统软件，例如引导程序、内核与文件系统等。

## 环境准备

编译主机要求：
- 操作系统：Ubuntu 16.04/18.04 或以上
- 存储空间：40G 的空闲磁盘空间
- 已安装好 Docker 且开发用户帐号有 docker 权限

## 下载 BSP SDK

- 到 [原厂 SDK](https://gitee.com/sophon-ai/bsp-sdk) 下载 `bsp-sdk-binary.tgz`
- 到 [下载中心] 下载 `bm1688_build.docker.tar`, `ec-a1688jd4_bsp.initial.bundle` 和 `ec-a1688jd4_bsp.update.bundle`

运行以下命令完成 BSP SDK 的初始化：

```shell
# 这里假设所有的下载文件都放在当前目录

# 安装开发工具
sudo apt install bison flex bc rsync kmod cpio sudo uuid-dev cmake libssl-dev fakeroot dpkg-dev device-tree-compiler u-boot-tools

# 从 ec-a1688jd4_bsp.initial.bundle 签出 BSP SDK
git init bm1688-sdk
cd bm1688-sdk
git fetch ../ec-a1688jd4_bsp.initial.bundle
git reset --hard FETCH_HEAD

# 从 ec-a1688jd4_bsp.update.bundle 更新到最新
git fetch ../ec-a1688jd4_bsp.update.bundle
git merge FETCH_HEAD

# 解压 SDK 包
tar xf ../bsp-sdk-binary.tgz

# 导入 Docker 编译映像
docker load < ../bm1688_build.docker.tar
```

## 更新 BSP SDK
- 到 [下载中心] 下载最新的 `ec-a1688jd4_bsp.update.bundle`

运行以下命令完成 BSP SDK 的更新：
```shell
# 这里假设所有的下载文件放在 BSP SDK 目录

# 从 bundle 拉取修改提交
git fetch ec-a1688jd4_bsp.update.bundle

# 查看修改记录
git log FETCH_HEAD

# 合并到当前开发分支
git merge FETCH_HEAD

# 移除 bundle 文件
rm ec-a1688jd4_bsp.update.bundle
```

## 编译升级固件

假设当前是 BSP SDK 目录，运行以下命令生成升级固件：
```shell
# 以下命令运行后需要输入用户密码来做 sudo 操作
./build.sh build_all

# 最终生成的固件文件为：install/soc_bm1688_asic/sdcard.zip
```

---

## Sophon SDK 开发


BM1688 的 SDK 开发依赖于 SOPHONSDK（一站式SDK），其中包含了模型转换、模型量化、算法移植等相关模块。提供了包括文档、视频、论坛、开源仓库等一系列资料帮助用户进行算法移植和开发工作。

目前系统固件适配 SOPHONSDK 的版本为 v1.7，参考以下链接即可获取 SDK 等相关的开发内容：

* [SDK 获取](https://developer.sophgo.com/site/index/material/89/all.html)

* [SDK使用手册](https://doc.sophgo.com/bm1688_sdk-docs/v1.7/docs_latest_release/docs/BM1688_CV186AH_SophonSDK_doc/common_test/disclaimer.html)

* [BSP开发参考手册](https://doc.sophgo.com/bm1688_sdk-docs/v1.7/docs_latest_release/docs/athena2-img/)



[下载中心]: https://community.t-firefly.com/doc/download/266