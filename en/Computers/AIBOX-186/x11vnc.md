# Remote Desktop X11VNC

x11vnc is a tool that allows administrators to connect to the server's real X desktop directly via VNC Viewer.

## Ubuntu
The Ubuntu system requires the manual installation of the X desktop environment. During the installation process, select lightdm as the Display Manager:
```
sudo apt update
sudo apt install -y xserver-xorg-video-dummy x11vnc xfce4
sudo vim /etc/X11/xorg.conf
sudo reboot
```
The contents of /etc/X11/xorg.conf are:
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
After the installation is complete, start the x11vnc service:

```shell
sudo x11vnc -display :0 -auth /var/lib/lightdm/.Xauthority &
```

You can then use the `$bm1688_ip:0` address to VNC remote connect from a PC.

The following is the specific operation process for VNC Viewer:

(1) Users first download [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) from the official website, selecting the appropriate installation package based on their PC system.

(2) After installation, open VNC Viewer and skip the login.

![](../../../bm1688_img/AIBOX-186/vnc-01.png)

(3) Enter the `$bm1688_ip:0` address of AIBOX-186:

![](../../../bm1688_img/AIBOX-186/ubuntu-vnc-02.png)

(4) Click the circular button next to the linaro username, select xfce, and enter the username and password, both are `linaro`:

![](../../../bm1688_img/AIBOX-186/ubuntu-vnc-03.png)

(5) Successfully enter the xfce4 desktop:

![](../../../bm1688_img/AIBOX-186/ubuntu-vnc-04.png)