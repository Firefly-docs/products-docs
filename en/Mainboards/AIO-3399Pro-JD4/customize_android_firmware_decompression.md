


* Extract the `release_update.img`

```
$ cd /path/to/your/firmware/dir
$ img_unpack AIO-3399ProJD4_Android8.1.0_HDMI_190509.img img
rom version: 6.0.1
build time: 2016-10-27 14:58:18
chip: 33333043
checking md5sum....OK
```

* Extract the `update.img`

```
$ cd img
$ afptool -unpack update.img update
Check file...OK
------- UNPACK -------
package-file	0x00000800	0x000002AC
Image/MiniLoaderAll.bin	0x00001000	0x0004394E
Image/parameter.txt	0x00045000	0x00000368
Image/trust.img	0x00045800	0x00400000
Image/uboot.img	0x00445800	0x00400000
Image/misc.img	0x00845800	0x0000C000
Image/resource.img	0x00851800	0x00038800
Image/kernel.img	0x0088A000	0x012D2014
Image/boot.img	0x01B5C800	0x0017893C
Image/recovery.img	0x01CD5800	0x00866B8C
Image/system.img	0x0253C800	0x41A9110C
Image/vendor.img	0x43FCE000	0x1206E0A0
Image/oem.img	0x5603C800	0x00058094
RESERVED	0x00000000	0x00000000
UnPack OK!
```

* View the file tree under the update directory

```
$ cd update/
$ tree
.
├── Image
│   ├── boot.img
│   ├── kernel.img
│   ├── MiniLoaderAll.bin
│   ├── misc.img
│   ├── oem.img
│   ├── parameter.txt
│   ├── recovery.img
│   ├── resource.img
│   ├── system.img
│   ├── trust.img
│   ├── uboot.img
│   └── vendor.img
├── loader.img
├── package-file
└── RESERVED

1 directory, 15 files
```
