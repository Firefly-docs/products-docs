# 编译第三方驱动

用户可能需要为自己添加的外设编译驱动，通常有两种方式：1）直接把驱动源码放入内核源码中编译。2）为了方便用户，我们也提供了 `linux-header` 和 `linux-image` 的 deb 包，方便
用户在设备上直接编译出驱动的 `ko` 文件。

## 安装

到[资源下载](https://community.t-firefly.com/doc/download/73)页面，下载` linux-header `和` linux-image `的 deb 包：

```
linux-4.4.185_4.4.185-8_arm64.changes
linux-headers-4.4.185_4.4.185-8_arm64.deb
linux-image-4.4.185_4.4.185-8_arm64.deb
linux-image-4.4.185-dbg_4.4.185-8_arm64.deb
linux-libc-dev_4.4.185-8_arm64.deb
```

把文件全部拷贝到设备上，通过` dpkg `进行安装：
```
dpkg -i ./*.deb
```

## 编译

由于` deb `是在 x86 上内核编译出来的，部分工具没有进行交叉编译，所以需要在设备上把这些工具编译一遍
```
cd /usr/src/linux-headers-x.x.xxx/
make headers_check
make headers_install
make scripts
# 可能会出错，但是不要紧
```

另外` module `也需要执行

```
depmod -a
```

如果在板子上编译驱动，`make install` 之后，也需要 `depmod -a`