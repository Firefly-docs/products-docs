## Compile Debian Firmware

This chapter introduces the compilation process of Debian firmware. It is recommended to develop under Ubuntu 18.04 system environment. If you use other system versions, you may need to adjust the compilation environment accordingly.

### Compile

The only difference between the compilation method of Debian firmware and Ubuntu is the use of different filesystems, other steps including configuration files are exactly the same.

So please refer to [Compile Ubuntu firmware](linux_compile.html#compile-ubuntu-firmware)

Now only introduce the difference: replacing of filesystem

#### Download Debian filesystem

* Download ：[Debian rootfs(64-bit)]()，and put it in the SDK

* Decompress

```bash
7z x Debian10-xxxx_RK3568_xxxx.7z
```

* Put rootfs under `ubuntu_rootfs/`

```bash
mkdir ubuntu_rootfs
mv Debian10-xxxx_RK3568_xxxx.img ubuntu_rootfs/
```

* Create a symbolic link to `rk356x_ubuntu_rootfs.img`

```bash
cd ubuntu_rootfs
ln -sf Debian10-xxxx_RK3568_xxxx.img rk356x_ubuntu_rootfs.img
```
**Then follow the normal steps just like you are compiling ubuntu firmware**