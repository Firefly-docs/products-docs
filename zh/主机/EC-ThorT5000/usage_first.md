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

