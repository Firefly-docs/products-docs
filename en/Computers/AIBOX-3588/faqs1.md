# jtop
```
sudo apt update
sudo apt install python3-pip
sudo pip3 install -U jetson-stats
```

# Important!Important!Important
<font color=red>Do not use `sudo apt-get upgrade` and `sudo apt-get dist-upgrade` .</font>
<br>
<font color=red>Do not use `sudo apt upgrade` and `sudo apt dist-upgrade` .</font>

To prevent being upgraded, you can comment out the official Jetson apt source of NVIDIA:

```
 cat /etc/apt/sources.list.d/nvidia-l4t-apt-source.list 
# SPDX-FileCopyrightText: Copyright (c) 2019-2024 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: LicenseRef-NvidiaProprietary
#
# NVIDIA CORPORATION, its affiliates and licensors retain all intellectual
# property and proprietary rights in and to this material, related
# documentation and any modifications thereto. Any use, reproduction,
# disclosure or distribution of this material and related documentation
# without an express license agreement from NVIDIA CORPORATION or
# its affiliates is strictly prohibited.

#deb https://repo.download.nvidia.com/jetson/common r36.4 main
#deb https://repo.download.nvidia.com/jetson/t234 r36.4 main
#deb https://repo.download.nvidia.com/jetson/ffmpeg r36.4 main
```

# nvpmodel 

||7W|15W|25W|MAXN_SUPER|
|----|----|----|----|----|
|Jetson Orin Nano 8GB|Mode ID 0|Mode ID 1|N/A|N/A|
|Jetson Orin Nano 8GB SUPER|N/A|Mode ID 0|Mode ID 1|Mode ID 2|

||MAXN_SUPER|MAXN|10W|15W|20W|40W|
|----|----|----|----|----|----|----|
|Jetson Orin NX 8GB|N/A|Mode ID 0|Mode ID 1|Mode ID 2|Mode ID 3|N/A|
|Jetson Orin NX 8GB SUPER|Mode ID 0|N/A|Mode ID 1|Mode ID 2|Mode ID 3|Mode ID 4|

||MAXN_SUPER|MAXN|10W|15W|25W|40W|
|----|----|----|----|----|----|----|
|Jetson Orin NX 16GB|N/A|Mode ID 0|Mode ID 1|Mode ID 2|Mode ID 3|N/A|
|Jetson Orin NX 16GB SUPER|Mode ID 0|N/A|Mode ID 1|Mode ID 2|Mode ID 3|Mode ID 4|

## Commands
* get power mode: `sudo nvpmodel -q`
* set power mode: `sudo nvpmodel -m <x>`
    * `<x>` is the power `Mode ID` (for example, 0, 1, 2, 3 or 4).

## GUI
![](../../../aibox_img/nvpmodel_gui.png)

* To switch the current power mode, click the NVIDIA icon to open a dropdown menu from the icon
* Click "Power mode" to open a submenu of power modes


# Browser
```
sudo apt update
sudo apt install chromium-browser
```

If the browser fails to open after installation, please follow the steps below: 

```
snap download snapd --revision=24724
sudo snap ack snapd_24724.assert
sudo snap install snapd_24724.snap
```


# HDMI Audio
`Settings` --> `Sound` --> `Output Device` --> `HDMI/ DisplayPort-Build-in Audio`

# Flash Images

<font color="red">**Notice:**</font>
* PC use X86 Ubuntu 22.04 or Ubuntu 20.04, do not use virtual machine.
* When flashing, AIBOX-3588 will format the internal storage device, please back up important data first.
* Orin NX power supply need **12V/5A**
* Orin NX carrier board hardware at least **V1.2**

## Download fireflyFlash.tbz2

## Unzip fireflyFlash.tbz2
```
mkdir fireflyFlash
tar xf fireflyFlash.tbz2 -C fireflyFlash
cd fireflyFlash
sudo ./l4t_flash_prerequisites.sh
```

## Put AIBOX-3588 into Force Recovery Mode
* Ensure that AIBOX-3588 is powered off
* Connect AIBOX-3588 and PC using TypeC
* Press and hold down AIBOX-3588 Recovery-Key
* AIBOX-3588 power on
* Release AIBOX-3588 Recovery-Key
* Check AIBOX-3588 Recovery Mode
    * use `lsusb` comand, AIBOX-3588 is in Force Recovery Mode if you see the message: `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.`
        * `<bbb>` any three-digit number
        * `<ddd>` any three-digit number
        * `<nnnn>` four-digit number
            * `7523` Jetson Orin Nano 8GB
            * `7423` Jetson Orin NX 8GB
            * `7323` Jetson Orin NX 16GB

## Flash
Enter these commands in the directory of `fireflyFlash`: `./firefly_flash.sh -d aibox`

<font color=red>Notice: After burning, the device must be on the desktop for at least 5 minutes before it can be powered off.</font>