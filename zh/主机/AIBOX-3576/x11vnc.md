# 远程桌面 X11VNC

x11vnc 是一个可以让管理员直接通过 VNC Viewer 来连接服务器的真实 X 桌面的工具。


## Ubuntu
Ubuntu 系统需要手动安装X桌面环境，安装过程中，Display Manager选择lightdm：
```
sudo apt update
sudo apt install -y xserver-xorg-video-dummy x11vnc xfce4
sudo vim /etc/X11/xorg.conf
sudo reboot
```
/etc/X11/xorg.conf中内容为：
```
Section "Device"
    Identifier  "Configured Video Device"
    Driver      "dummy"
    VideoRam 256000
EndSection

Section "Monitor"
    Identifier  "Configured Monitor"
    HorizSync 5.0 - 1000.0
    VertRefresh 5.0 - 200.0
    ModeLine "1920x1080" 148.50 1920 2448 2492 2640 1080 1084 1089 1125 +Hsync +Vsync
#    Modeline "1280x800" 24.15 1280 1312 1400 1432 800 819 822 841
EndSection

Section "Screen"
    Identifier  "Default Screen"
    Monitor     "Configured Monitor"
    Device      "Configured Video Device"
    DefaultDepth 24
    SubSection "Display"
    Depth 24
    Modes "1920x1080"
#    Modes "1280x800"
    EndSubSection
EndSection
```
安装完成后，启动x11vnc服务：

```shell
sudo x11vnc -display :0 -auth /var/lib/lightdm/.Xauthority &
```

然后就可以在 PC 机上用 `$bm1688_ip:0` 地址来 VNC 远程连接。

以下是 VNC Viewer 的具体操作过程：

（1）用户首先到官方下载 [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/)，根据自身 PC 机系统选择对应安装包进行下载并安装。

（2）安装完成后打开 VNC Viewer，跳过登录

![](../../../aibox_img/AIBOX-3576/vnc-01.png)

（3）输入 AIBOX-3576 的 `$bm1688_ip:0` 地址：

![](../../../aibox_img/AIBOX-3576/ubuntu-vnc-02.png)

（4）点击linao用户名右侧的圆形按钮，选择xfce， 输入用户与密码，均为 `linaro`：

![](../../../aibox_img/AIBOX-3576/ubuntu-vnc-03.png)

（5）成功进入 xfce4 桌面：

![](../../../aibox_img/AIBOX-3576/ubuntu-vnc-04.png)


