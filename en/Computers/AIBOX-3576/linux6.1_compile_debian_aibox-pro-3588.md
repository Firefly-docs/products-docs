### rootfs
* Download root filesystem: [Debian Rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/333.html), please use the filesystem from the `kernel-6.1` directory.
* Extract rootfs and link it

#### RK3588
```
# Extract
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the extracted rootfs image to SDK and create a symbolic link
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

### Configure
#### Core-3588JD4

```
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_debian_defconfig
```

### Build
```
./build.sh all
```

The generated firmware is in the `output/update/` directory, e.g., `AIBOX-PRO-G2-3588JD4_Debian.XXX.img`
