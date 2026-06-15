## 编译 Debian 固件

本章介绍 Debian 固件的编译流程，推荐在 Ubuntu 18.04 系统环境下进行开发，若使用其它系统版本，可能需要对编译环境做相应调整。

### 编译

Debian 固件的编译方法和 Ubuntu 的唯一区别是使用的文件系统不一样，其他步骤包括配置文件完全一致

因此请参考 [编译 Ubuntu 固件](linux_compile.html#bian-yi-ubuntu-gu-jian)

下面只介绍不同点，文件系统的替换

#### 下载 Debian 根文件系统

* 下载根文件系统：[Debian 根文件系统(64位)]()，放到 SDK 路径下

* 解压文件

```bash
7z x Debian10-xxxx_RK3568_xxxx.7z
```

* 将根文件系统放到 `ubuntu_rootfs/` 目录下

```bash
mkdir ubuntu_rootfs
mv Debian10-xxxx_RK3568_xxxx.img ubuntu_rootfs/
```

* 创建一个链接将文件系统链接到 `rk356x_ubuntu_rootfs.img`

```bash
cd ubuntu_rootfs
ln -sf Debian10-xxxx_RK3568_xxxx.img rk356x_ubuntu_rootfs.img
```
**之后按照正常编译 Ubuntu 固件的步骤执行即可**