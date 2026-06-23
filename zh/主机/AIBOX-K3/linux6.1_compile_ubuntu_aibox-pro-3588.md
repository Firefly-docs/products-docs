### rootfs
* 下载根文件系统：[Ubuntu 根文件系统(64位) Kernel6.1](https://www.t-firefly.com/doc/download/290.html)，请使用网盘中 kernel-6.1 目录下的文件系统。
* 解压 rootfs 并链接 rootfs

#### RK3588 
```
# 解压
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

### 配置
### RK3588

```
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_ubuntu_defconfig
```

### 编译
```
./build.sh all
```

生成的固件在 `output/update/` 目录下，比如 ``AIBOX-PRO-G2-3588JD4_Ubuntu.XXX.img`
