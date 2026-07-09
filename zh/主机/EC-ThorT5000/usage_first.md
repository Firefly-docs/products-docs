# 初次使用
## 登录
用户名： `nvidia`<br>
密码： `nvidia`

## Jetpack 


| |OS|Kernel|CUDA|
|----|----|----|----|
|JetPack 7|Ubuntu 24.04|6.8|13|


由于 jetpack 的软件栈组件较大，默认没有安装，需要您手动安装。

```
sudo apt update
sudo apt install nvidia-jetpack
```

## jtop
```
sudo apt update
sudo apt install python3-pip
sudo pip3 install -U jetson-stats
```

## 重要!重要!重要
<font color=red>不要执行 `sudo apt-get upgrade`和 `sudo apt-get dist-upgrade`。</font>
<br>
<font color=red>不要执行 `sudo apt upgrade` 和 `sudo apt dist-upgrade`。</font>

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

