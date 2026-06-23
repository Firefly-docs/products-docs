# Update Firmware

## Preparation before upgrading

PC OS: Ubuntu22.04, support for NFS services is required, and there may be some missing commands during the upgrade process, such as' sshpass', which users need to install themselves.


## Put into recovery mode


When the device is powered off, use a small ejector pin or toothpick to push it in through the headphone jack, which will press the Recovery button. While pressing the Recovery button, the device will be powered on and held for 2 seconds before releasing it.


Connect the device to the computer using a type-c cable. Assuming the device successfully enters Recovery mode, it can be seen in Linux:

```
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0b05:19af ASUSTek Computer, Inc. AURA LED Controller
Bus 001 Device 004: ID 05e3:0608 Genesys Logic, Inc. Hub
Bus 001 Device 022: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
Bus 001 Device 023: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 001 Device 030: ID 0955:7523 NVIDIA Corp. APX
Bus 001 Device 040: ID 046d:c31c Logitech, Inc. Keyboard K120
Bus 001 Device 086: ID 067b:2731 Prolific Technology, Inc. USB SD Card Reader     
Bus 001 Device 119: ID 067b:23a3 Prolific Technology, Inc. ATEN Serial Bridge
Bus 001 Device 120: ID 2109:2817 VIA Labs, Inc. USB2.0 Hub             
Bus 001 Device 121: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 123: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```

Among them, `0955:7523 NVIDIA Corp. APX` indicates that the computer has detected NVIDIA devices that have entered Recovery mode.

Notice: If do not found `0955:7523 NVIDIA Corp. APX`, board need to change to `OTG` by `TYPE C-SEL`.

* `0955:7523 NVIDIA Corp. APX` : Jetson Orin Nano 8GB
* `0955:7423 NVIDIA Corp. APX` : Jetson Orin NX 8GB
* `0955:7323 NVIDIA Corp. APX` : Jetson Orin NX 16GB

## R36.3 (JetPack 6.0)
### Download Firmware

You can directly download it from Firefly [Download Page](https://www.t-firefly.com/doc/download/266.html)

After downloading, perform tar decompression:

After decompression, enter the firmware package for upgrading



If everything goes smoothly, after a successful upgrade, the following fields will appear to indicate that the upgrade is successful:
```
Copied 16896 bytes from /mnt/internal/gpt_secondary_3_0.bin to address 0x03ffbe00 in flash
[ 233]: l4t_flash_from_kernel: Successfully flash the qspi
[ 233]: l4t_flash_from_kernel: Flashing success
[ 233]: l4t_flash_from_kernel: The device size indicated in the partition layout xml is smaller than the actual size. This utility will try to fix the GPT.
Flash is successful
Reboot device
Cleaning up...
Log is saved to Linux_for_Tegra/initrdlog/flash_1-2_0_20250527-153418.log
```

## R36.4 (JetPack 6.2)

* Support Super Mode 
* Orin NX power supply need **12V/5A**
* Orin NX carrier board hardware at least **V1.1**

### Download fireflyFlash.tbz2
[Download](https://www.t-firefly.com/doc/download/266.html)
<br>
`Firmware` --> `Jetson Linux`

### Unzip fireflyFlash.tbz2
```
mkdir fireflyFlash
tar xf fireflyFlash.tbz2 -C fireflyFlash
cd fireflyFlash
sudo ./l4t_flash_prerequisites.sh
```

### Flash
Enter these commands in the directory of `fireflyFlash`: `./firefly_flash.sh -d aio-orin`

<font color=red>Notice: After burning, the device must be on the desktop for at least 5 minutes before it can be powered off.</font>
