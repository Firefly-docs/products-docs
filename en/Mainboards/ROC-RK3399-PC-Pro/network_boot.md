## Network booting

Network booting is to use TFTP to download the kernel and dtb files from the server to the memory of the target machine. At the same time, you can use NFS to mount the network root file system to the target machine to achieve diskless booting of the target machine. The following is an example based on Firefly-RK3399 for user reference.

Ready to work:
* Firefly-RK3399 device;
* Router, network cable;
* Server with NFS and TFTP installed;
* A created root file system.

Note: In the example, the Ubuntu system PC is used as the server, and the connection with the device is realized through a router and a network cable. Users can make adjustments according to their actual conditions, but if the PC is directly connected to the device, please use a crossover cable. Please make sure that the server and the target machine are in the same LAN.

### Server deployment
1.Deploy TFTP service on the server:

Install TFTP service:
```
sudo apt-get install tftpd-hpa
```

Create the `/tftpboot` directory and grant permissions:
```
mkdir /tftpboot
sudo chmod 777 /tftpboot
```

Then modify the configuration file `/etc/default/tftpd-hpa` of the TFTP server, and modify the content as follows:
```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot" #This is set according to the user's actual tftp directory
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-c -s -l"
```

Restart the TFTP server after exiting the modification:
```
sudo service tftpd-hpa restart
```

Create a file in the `/tftpboot` directory to test whether the TFTP service is available:
```bash
firefly@Desktop:~$ cd /tftpboot/
firefly@Desktop:/tftpboot$ touch test
firefly@Desktop:/tftpboot$ cd /tmp
firefly@Desktop:/tmp$ tftp 127.0.0.1
tftp> get test
tftp> q
```
Check the files in the `/tmp` directory. If you see the `test` file, it means the TFTP server is available.

2.Deploy the NFS service on the server:

Install the NFS server:

```
sudo apt-get install nfs-kernel-server
```

Create a shared directory:

```
sudo mkdir /nfs
sudo chmod 777 /nfs
cd /nfs
sudo mkdir rootfs
sudo chmod 777 rootfs
```

Then copy the created root file system to the `/nfs/rootfs` directory.

Then configure NFS and add the location of the shared directory in `/etc/exports`:

```bash
/nfs/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

The shared directory is set according to the actual situation of the user, and the "*" represents that all users can access.

Restart the NFS server:

```
sudo /etc/init.d/nfs-kernel-server restart
```

Mount the shared directory locally to test whether the NFS server is available:

```
sudo mount -t nfs 127.0.0.1:/nfs/rootfs/ /mnt
```

If the contents consistent with `/nfs/rootfs` are viewed in the `/mnt` directory, it indicates that the NFS server is deployed successfully. Then you can unmount:

```
sudo umount /mnt
```

### Kernel configuration

If you want to mount the network root file system, you need to configure the kernel and modify the configuration of the root file system in dts.

First perform the kernel configuration, execute `make menuconfig` in the kernel directory, and select the relevant configuration:

```
[*] Networking support --->
         Networking options --->
                [*] IP: kernel level autoconfiguration
                [*] IP: DHCP support
                [*] IP: BOOTP support
                [*] IP: RARP support


File systems --->
        [*] Network File Systems --->
                [*] Root file system on NFS
```

Modify the `rk3399-firefly.dts` configuration, modify the `bootargs` parameter under the `chosen` node in the dts file, and choose to use NFS to mount the remote root file system. The content is as follows.

```bash
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 root=/dev/nfs rootfstype=ext4 rootwait";
};

# Mainly to modify the value of root, root=/dev/nfs means to mount the network root file system through NFS
```

Compile the kernel:

```
make ARCH=arm64 rk3399-firefly.img -j12
```

After the compilation is complete, copy the compiled kernel files `boot.img` and `rk3399-firefly.dtb` files to the `/tftpboot` directory:

```
cp boot.img /tftpboot
cp arch/arm64/boot/dts/rockchip/rk3399-firefly.dtb /tftpboot
```

For detailed instructions, please refer to `kernel/Documentation/filesystems/nfs/nfsroot.txt` in the kernel directory


### U-Boot Settings

Please make sure that the network cable of the target machine is plugged in and connected to the LAN of the server.

The target machine boots into U-Boot command line mode, and set the following parameters:

```bash
=> setenv ipaddr 192.168.31.101 # Set the IP address of the target machine
=> setenv serverip 192.168.31.106 # Set serverip as the server IP address

# Set to download the kernel and dtb file from TFTP to the corresponding address, users should modify the corresponding address according to their actual target machine
=> setenv bootcmd tftpboot 0x0027f800 boot.img \; tftpboot 0x08300000 rk3399-firefly.dtb \; bootm 0x0027f800-0x08300000

# Set and mount the network root file system, the IP parameters are in order: target machine IP: server IP: gateway: netmask: device name: off, you can set ip=dhcp more simply, and automatically assign IP through DHXP
=> setenv bootargs root=/dev/nfs rw nfsroot=192.168.31.106:/nfs/rootfs,v3 ip=192.168.31.101:192.168.31.106:192.168.31.1:255.255.255.0::eth0:off

# If you don’t want to set it every time you boot, you can save the variables configured above
=> saveenv
Saving Environment to MMC...
Writing to MMC(0)... done

# Start the target machine
=> boot
ethernet@fe300000 Waiting for PHY auto negotiation to complete. done
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename'boot.img'.
Load address: 0x27f800
Loading: ############################################# ################
         ############################################## ###############
         ############################################## ###############
         ############################################## ###############
         ############################################## ###############
         ##############################################
         475.6 KiB/s
done
Bytes transferred = 20072448 (1324800 hex)
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename'rk3399-firefly.dtb'.
Load address: 0x8300000
Loading: #######
         645.5 KiB/s
done
Bytes transferred = 97212 (17bbc hex)
## Booting Android Image at 0x0027f800 ...
Kernel load addr 0x00280000 size 19377 KiB
## Flattened Device Tree blob at 08300000
   Booting using the fdt blob at 0x8300000
   XIP Kernel Image ... OK
   Loading Device Tree to 0000000073edc000, end 0000000073ef6bbb ... OK
Adding bank: 0x00200000-0x08400000 (size: 0x08200000)
Adding bank: 0x0a200000-0x80000000 (size: 0x75e00000)
Total: 912260.463 ms

Starting kernel ...

...
```

Visible in the boot kernel log:

```
[12.146297] VFS: Mounted root (nfs filesystem) on device 0:16.
```

It indicates that the network root file system has been mounted.

Precautions

* Ensure that the TFTP server and NFS server are available;
* Make sure that the target machine is plugged into the network cable first and then turned on, and is in the same LAN as the server. If it is directly connected to the target machine and the server, please use a crossover network cable;
* In the kernel configuration, `Root file system on NFS` depends on the `IP: kernel level autoconfiguration` option, please select `IP: kernel level autoconfiguration` first, and then you can find the `Root file system on NFS` option;
* In the U-Boot command line, please confirm the burning address of `boot.img` and the burning address of dtb file. (Hint: In the file structure of `boot.img`, there is a 2k header file at the beginning, and then the kernel. So when TFTP downloads the kernel to the target machine, it must be downloaded to the corresponding kernel address minus 0x0800);
* When setting to mount the remote root file system, `v3` in `nfsroot=192.168.31.106:/nfs/rootfs,v3` represents the NFS version information, please add it to avoid the problem of unsuccessful mounting.