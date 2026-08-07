# First Use
## Login
User: `nvidia`<br>
Password: `nvidia`

## Jetpack


| |OS|Kernel|CUDA|
|----|----|----|----|
|JetPack 7|Ubuntu 24.04|6.8|13|


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

