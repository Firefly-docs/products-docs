### rootfs
* 下载根文件系统：[Debian 根文件系统(64位) Kernel6.1](https://www.t-firefly.com/doc/download/290.html)，请使用网盘中 kernel-6.1 目录下的文件系统。
* 解压 rootfs 并链接 rootfs

#### RK3588 
```
# 解压
7z x debian12_xxxx_rootfs_xxxx.7z

# 将解压后的 rootfs 镜像移动到 sdk 并创建一个符号链接
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

### 配置
#### Core-3588JD4

```
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_debian_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 `AIBOX-PRO-G2-3588JD4_Debian.XXX.img` 
