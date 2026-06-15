## 导出设备系统

此章节适用于:
当用户已经在一台设备上完成工作环境的部署，需要将当前环境完整导出，以批量部署到其它同平台设备上。用户也可以通过导出设备文件系统来备份当前的开发环境。

导出设备系统分为两步
1. 导出设备rootfs
2. 二次打包完整固件，在PC端将rootfs与其它固件组成部分二次打包，生成完整固件

### 导出设备rootfs
1. 在设备端安装 `fireflydev`

	```
	sudo apt update
	sudo apt install fireflydev
	```

2. 使用`ff_export_rootfs`导出根文件系统

	* 建议使用容量较大的移动硬盘

	* 导出工具会执行apt clean等操作以减小文件系统大小

	* 将根文件系统导出到`/media/firefly/AC91-C4AE/`目录下，实际例子如下：
	```
        root@firefly:~# ff_export_rootfs
        ff_export_rootfs </path/to/store> [-t <ext4|btrfs>]

	ff_export_rootfs /media/firefly/AC91-C4AE/
	```

    * 压缩文件系统，删除不必要的空白空间以减少存储器资源的占用
    ```
        #有部分客户说导出的rootfs大小为3.3G可实际只用了3G，原因是没有对rootfs进行压缩
        e2fsck -p -f Firefly_Ubuntu_18.04.6_rootfs.img
        resize2fs  -M Firefly_Ubuntu_18.04.6_rootfs.img
    ```

### 二次打包完整固件
* 工具下载：[firefly-linux-repack](http://www.t-firefly.com/doc/download/page/id/54.html#other_396)
* 本工具适用于用户需要替换固件中任意一组成部分
* 在PC端安装必要环境：`sudo apt-get install lib32stdc++6`


```
firefly-linux-repack
    ├── bin
    │   ├── afptool
    │   └── rkImageMaker
    ├── pack.sh
    ├── Readme_en.md
    ├── Readme.md
    └── unpack.sh

```
1. 解包
    把官方发布的固件拷贝到当前目录，重命名为update.img , 执行unpack.sh
    解包完成后，生成的文件在output目录下.

2. 合包
    保持当前目录结构，文件名等不变，用客户自己的文件替换output/下同名的文件
    执行pack.sh, 执行完后，生成new_update.img，即为打包好的固件
	rootfs文件名必须为rootfs.img
	parameter.txt文件名必须为parameter.txt

* 注意：
	合包过程中，如果rootfs分区不是最后一个分区，那么程序会根据rootfs文件的大小，
	自动修改parameter.txt中rootfs分区的大小。
	如果用户自己有改动parameter.txt，请留意整个合包的流程。

```
tar -xzf firefly-linux-repack.tgz
cd firefly-linux-repack
mv /xxx/FIREFLY-RK3399-UBUNTU18.04-GPT-20200714-1510.img update.img
./unpack.sh

cp /customer/rootfs.img  output/Image/rootfs.img

./pack.sh
ls new_update.img
```