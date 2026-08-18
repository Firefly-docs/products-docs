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
* When flashing, AIBOX-Orin Nano will format the internal storage device, please back up important data first.
* Orin NX power supply need **12V/5A**
* Orin NX carrier board hardware at least **V1.2**

## Download fireflyFlash.tbz2
[Download](https://community.t-firefly.com/en/download/236)
`Firmware` --> `Jetson Linux`

## Unzip fireflyFlash.tbz2
```
mkdir fireflyFlash
tar xf fireflyFlash.tbz2 -C fireflyFlash
cd fireflyFlash
sudo ./l4t_flash_prerequisites.sh
```

## Put AIBOX-Orin Nano into Force Recovery Mode
* Ensure that AIBOX-Orin Nano is powered off
* Connect AIBOX-Orin Nano and PC using TypeC
* Press and hold down AIBOX-Orin Nano Recovery-Key
* AIBOX-Orin Nano power on
* Release AIBOX-Orin Nano Recovery-Key
* Check AIBOX-Orin Nano Recovery Mode
    * use `lsusb` comand, AIBOX-Orin Nano is in Force Recovery Mode if you see the message: `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.`
        * `<bbb>` any three-digit number
        * `<ddd>` any three-digit number
        * `<nnnn>` four-digit number
            * `7523` Jetson Orin Nano 8GB
            * `7423` Jetson Orin NX 8GB
            * `7323` Jetson Orin NX 16GB

## Flash
Enter these commands in the directory of `fireflyFlash`: `./firefly_flash.sh -d aibox`

<font color=red>Notice: After burning, the device must be on the desktop for at least 5 minutes before it can be powered off.</font>