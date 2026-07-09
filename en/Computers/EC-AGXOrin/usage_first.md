# First Use
## Login
User: `nvidia`<br>
Password: `nvidia`

## Jetpack

| |OS|Kernel|CUDA|
|----|----|----|----|
|JetPack 6|Ubuntu 22.04|5.15|12|



The following commands will install all other JetPack components that correspond to your version of Jetson Linux L4T:

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

## Important!Important!Important
<font color=red>Do not use `sudo apt-get upgrade` and `sudo apt-get dist-upgrade`.</font>
<br>
<font color=red>Do not use `sudo apt upgrade` and `sudo apt dist-upgrade`.</font>

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

