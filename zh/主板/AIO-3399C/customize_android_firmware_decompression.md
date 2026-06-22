* 解压 `release_update.img`

```
$ cd /path/to/your/firmware/dir
$ img_unpack Firefly-RK3399_20161027.img img
rom version: 6.0.1
build time: 2016-10-27 14:58:18
chip: 33333043
checking md5sum....OK
```

* 解压 `update.img`

```
$ cd img
$ afptool -unpack update.img update
Check file...OK
------- UNPACK -------
package-file	0x00000800	0x00000280
Image/MiniLoaderAll.bin	0x00001000	0x0003E94E
Image/parameter.txt	0x00040000	0x00000350
Image/trust.img	0x00040800	0x00400000
Image/uboot.img	0x00440800	0x00400000
Image/misc.img	0x00840800	0x0000C000
Image/resource.img	0x0084C800	0x0003FE00
Image/kernel.img	0x0088C800	0x00F5D00C
Image/boot.img	0x017EA000	0x0014AD24
Image/recovery.img	0x01935000	0x013C0000
Image/system.img	0x02CF5000	0x2622A000
RESERVED	0x00000000	0x00000000
UnPack OK!
```

* 查看 update 目录下的文件树

```
$ cd update/
$ tree
.
├── Image
│   ├── boot.img
│   ├── kernel.img
│   ├── MiniLoaderAll.bin
│   ├── misc.img
│   ├── parameter.txt
│   ├── recovery.img
│   ├── resource.img
│   ├── system.img
│   ├── trust.img
│   └── uboot.img
├── package-file
└── RESERVED

1 directory, 12 files
```



