## Export device system

This chapter applies to:
When the user has completed the deployment of the work environment on one device, the current environment needs to be completely exported for batch deployment to other devices on the same platform.Users can also backup the current development environment by exporting the device file system.

The export device system is divided into two steps
1. Export device rootfs
2. Re-package the complete firmware, re-package rootfs and other firmware components on the PC, and generate complete firmware

### Export device rootfs
1. Install `fireflydev` on the device

```
sudo apt update
sudo apt install fireflydev
```

2. Use `ff_export_rootfs` to export the root file system

* It is recommended to use a mobile hard disk with a larger capacity

* The export tool will perform operations such as apt clean to reduce the file system size

* Export the root file system to the `/media/firefly/AC91-C4AE/` directory, the actual example is as follows:
    ```
    root@firefly:~# ff_export_rootfs
            ff_export_rootfs </path/to/store> [-t <ext4|btrfs>]

    ff_export_rootfs /media/firefly/AC91-C4AE/
    ```

* Compresses the root file system and removes unnecessary empty space to reduce memory usage
    ```
        #Some customers say that the exported rootfs size is 3.3G, but only 3G is actually used, because the rootfs is not compressed
        e2fsck -p -f Firefly_Ubuntu_18.04.6_rootfs.img
        resize2fs  -M Firefly_Ubuntu_18.04.6_rootfs.img
    ```

### Second package complete firmware
* Tool download: [firefly-linux-repack](http://en.t-firefly.com/doc/download/52.html#other_349)
* This tool is suitable for users who need to replace any part of the firmware349
* Install the necessary environment on the PC: `sudo apt-get install lib32stdc++6`

```
firefly-linux-repack
    ├── bin
    │ ├── afptool
    │ └── rkImageMaker
    ├── pack.sh
    ├── Readme_en.md
    ├── Readme.md
    └── unpack.sh

```
1. Unpack
    Copy the officially released firmware to the current directory, rename it to update.img, execute unpack.sh
    After unpacking is complete, the generated file is in the output directory.

2. Combined package
    Keep the current directory structure, file name, etc. unchanged, replace the file with the same name under output/ with the customer's own file
    Execute pack.sh, after execution, generate new_update.img, which is the packaged firmware
The rootfs file name must be rootfs.img
The file name of parameter.txt must be parameter.txt

* Note:
During the package process, if the rootfs partition is not the last partition, the program will follow the size of the rootfs file,
Automatically modify the size of the rootfs partition in parameter.txt.
If the user himself changes parameter.txt, please pay attention to the entire package process.

```
tar -xzf firefly-linux-repack.tgz
cd firefly-linux-repack
mv /xxx/FIREFLY-RK3399-UBUNTU18.04-GPT-20200714-1510.img update.img
./unpack.sh

cp /customer/rootfs.img output/Image/rootfs.img

./pack.sh
ls new_update.img
```