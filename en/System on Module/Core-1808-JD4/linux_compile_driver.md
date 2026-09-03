# Compiling drivers

Users may need to compile the driver for the peripherals. There are usually two ways: 1) Put the driver source code into the kernel source code to compile. 2) For the convenience of users, we also provide the DEB package of` Linux header `and` Linux image `to facilitate users to compile the` ko `file of the driver directly on the device.

## Install

Go to the resource [download page](https://community.t-firefly.com/en/doc/download/83) and download the DEB packages of `Linux header` and `Linux image`:

```
linux-4.4.185_4.4.185-8_arm64.changes
linux-headers-4.4.185_4.4.185-8_arm64.deb
linux-image-4.4.185_4.4.185-8_arm64.deb
linux-image-4.4.185-dbg_4.4.185-8_arm64.deb
linux-libc-dev_4.4.185-8_arm64.deb
```

Copy all the files to the device and install them through `dpkg`：

```
dpkg -i ./*.deb
```

## Compile

Because 'DEB' is compiled on the `x86` host, so some tools need to be compiled on the device

```
cd /usr/src/linux-headers-x.x.xxx/
make headers_check
make headers_install
make scripts
# It may go wrong, but it doesn't matter
```

In addition, `module` also needs to be executed：

```
depmod -a
```

If the driver is compiled on the board, after `make install`, you also need `depmod - a`
