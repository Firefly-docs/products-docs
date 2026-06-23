### rootfs
* Download root filesystem: [Ubuntu Rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/333.html), please use the filesystem from the `kernel-6.1` directory.
* Extract rootfs and link it

#### RK3588
```
# Extract
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

### Configure
### RK3588

```
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_ubuntu_defconfig
```

### Build
```
./build.sh all
```

The generated firmware is in the `output/update/` directory, e.g., `AIBOX-PRO-G2-3588JD4_Ubuntu.XXX.img`
