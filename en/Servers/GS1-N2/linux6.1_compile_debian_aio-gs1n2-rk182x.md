### Rootfs
* Download rootfs here [Debian rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/140.html), please use rootfs under kernel-6.1 folder.
* Decompress rootfs and create a symbolic link

#### RK3588 
```
# Decompress
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the rootfs image to sdk and then create a symbolic link.
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

#### RK3576 
```
# Decompress
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the rootfs image to sdk and then create a symbolic link.
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3576_debian_rootfs.img
cd ..
```

### Config
#### RK3588
##### Core-3588JD4

```
./build.sh firefly_rk3588_aio-gs1n2-3588jd4-rk182x_debian_defconfig
```

##### Core-3588SJD4 AI

```
./build.sh firefly_rk3588_aio-gs1n2-3588sjd4-ai-rk182x_debian_defconfig
```

#### RK3576
##### Core-3576JD4

```
./build.sh firefly_rk3576_aio-gs1n2-3576jd4-rk182x_debian_defconfig
```

### Build
```
./build.sh all
```

The generated firmware at `output/update/` , eg: `AIO-GS1N2-3588JD4-RK182X_Debian.XXX.img`
