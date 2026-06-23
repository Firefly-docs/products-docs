# jtop
```
sudo apt update
sudo apt install python3-pip
sudo pip3 install -U jetson-stats
```

# 重要!重要!重要
<font color=red>不要执行 `sudo apt-get upgrade` 和 `sudo apt-get dist-upgrade` 。</font>
<br>
<font color=red>不要执行 `sudo apt upgrade` 和 `sudo apt dist-upgrade` 。</font>

为了防止被 upgrade，可以注释掉英伟达官方的 jetson apt 源： 

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

## 命令
* 读取: `sudo nvpmodel -q`
* 设置: `sudo nvpmodel -m <x>`
    * `<x>` 是 `Mode ID` (比如, 0, 1, 2, 3 or 4)

## GUI
![](../../../aibox_img/nvpmodel_gui.png)

* 点击英伟达的图标
* 点击"Power mode", 选择所需的功率

# Browser
```
sudo apt update
sudo apt install chromium-browser
```

如果安装浏览器后，打开失败，进行如下操作:

```
snap download snapd --revision=24724
sudo snap ack snapd_24724.assert
sudo snap install snapd_24724.snap
```


# HDMI Audio
`Settings` --> `Sound` --> `Output Device` --> `HDMI/ DisplayPort-Build-in Audio`


# 烧录固件

<font color="red">**注意：**</font>
* PC 端推荐 X86 Ubuntu 22.04 或者 Ubuntu 20.04，不推荐使用虚拟机。
* 烧录过程中会格式化 AIBOX-3576 内置的存储设备，重要数据请先备份。
* Orin NX 电源适配器需要 **12V/5A**
* Orin NX 的底板硬件版本至少是 **V1.2**

## 下载 fireflyFlash.tbz2

## 解压 fireflyFlash.tbz2
```
mkdir fireflyFlash
tar xf fireflyFlash.tbz2 -C fireflyFlash
cd fireflyFlash
sudo ./l4t_flash_prerequisites.sh
```

## AIBOX-3576 进入烧录模式
* AIBOX-3576 先断电
* 使用 Type-C 数据线连接 AIBOX-3576 的 OTG 口和 PC 端
* 长按 AIBOX-3576 的 Recovery 键
* AIBOX-3576 供电
* 释放 AIBOX-3576 的 Recovery 键
* 检查 AIBOX-3576 是否进入 Recovery 模式
    * 使用 `lsusb` 命令， 当你看到 `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.` 时，即进入了 Recovery 模式。
        * `<bbb>` 任何三位数
        * `<ddd>` 任何三位数
        * `<nnnn>` 四位数
            * `7523` Jetson Orin Nano 8GB
            * `7423` Jetson Orin NX 8GB
            * `7323` Jetson Orin NX 16GB

## 烧录
在 `fireflyFlash` 目录下，执行命令：  `./firefly_flash.sh -d aibox`  

<font color=red>注意：烧录完，设备进入桌面后至少 5 分钟才能断电。</font>
