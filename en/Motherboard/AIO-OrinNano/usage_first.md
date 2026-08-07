# First Use

## Important!Important!Important
<font color=red>Do not use `sudo apt-get upgrade` and `sudo apt-get dist-upgrade` .</font>
<br>
<font color=red>Do not use `sudo apt upgrade` and `sudo apt dist-upgrade` .</font>


To prevent automatic upgraded : 
* Modify `/etc/apt/apt.conf.d/20auto-upgrades` (if this file doesn’t exist, please create it)

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

* Disable and stop apt-daily timers

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


The following commands will install all other JetPack components that correspond to your version of Jetson Linux L4T:
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

## browser
If the browser fails to open after installation, please follow the steps below: 

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

### Commands
* get power mode: `sudo nvpmodel -q`
* set power mode: `sudo nvpmodel -m <x>`
    * `<x>` is the power `Mode ID` (for example, 0, 1, 2, 3 or 4).

### GUI
![](../../../bm1688_img/nvpmodel_gui.png)

* To switch the current power mode, click the NVIDIA icon to open a dropdown menu from the icon
* Click “Power mode” to open a submenu of power modes


