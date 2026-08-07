# 初次使用

## 重要!重要!重要
<font color=red>不要执行 `sudo apt-get upgrade` 和 `sudo apt-get dist-upgrade` 。</font>
<br>
<font color=red>不要执行 `sudo apt upgrade` 和 `sudo apt dist-upgrade` 。</font>



为了防止被自动 upgrade : 
* 修改 `/etc/apt/apt.conf.d/20auto-upgrades` (如果文件不存在，就创建它)

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

* 关闭和停止 apt-daily timers 服务

```
sudo systemctl disable apt-daily.timer
sudo systemctl disable apt-daily-upgrade.timer
sudo systemctl stop apt-daily.timer
sudo systemctl stop apt-daily-upgrade.timer
```

## Jetpack 

| |OS|Kernel|CUDA|
|----|----|----|----|
|JetPack 6|Ubuntu 22.04|5.15|12|

由于 jetpack 的软件包较大，默认没有安装，需要您手动安装。

```
sudo apt update
sudo apt install nvidia-jetpack
sudo apt-cache show nvidia-jetpack
```

## jtop
```
sudo apt update
sudo apt install python3-pip
sudo pip3 install -U jetson-stats
```

## 浏览器
如果安装浏览器后，打开失败，进行如下操作:

```
snap download snapd --revision=24724
sudo snap ack snapd_24724.assert
sudo snap install snapd_24724.snap
```

## nvpmodel 

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

### 命令
* 读取: `sudo nvpmodel -q`
* 设置: `sudo nvpmodel -m <x>`
    * `<x>` 是 `Mode ID` (比如, 0, 1, 2, 3 or 4)

### GUI
![](../../../bm1688_img/nvpmodel_gui.png)

* 点击英伟达的图标
* 点击"Power mode", 选择所需的功率

