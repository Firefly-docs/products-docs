# Remote Desktop X11VNC

x11vnc is a tool that allows administrators to connect to the server's real X desktop directly through the VNC Viewer.
## Debian
AIO-1684JD4 has installed the xfce4 desktop and configured the x11vnc related environment by default. The user directly executes the following command to wait for the connection:

```shell
sudo x11vnc -display :0 -auth /var/lib/lightdm/.Xauthority &
```

Then you can use `$bm1684_ip:0` address to VNC remote connection on the PC.

The following is the specific operation process of VNC Viewer:

(1) The user first downloads [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) from the official website, and selects the corresponding installation package according to the PC system to download and install.

(2) After the installation is complete, open VNC Viewer and skip login:

![](../../../bm1684_img/AIO-1684JD4/vnc-01.png)

(3) Enter `$bm1684_ip:0` address of AIO-1684JD4:

![](../../../bm1684_img/AIO-1684JD4/vnc-02.png)

(4) Enter the user and password, both `linaro`:

![](../../../bm1684_img/AIO-1684JD4/vnc-03.png)

(5) Successfully entered the xfce4 desktop:

![](../../../bm1684_img/AIO-1684JD4/vnc-04.png)

## Ubuntu
To use the X Desktop environment on Ubuntu system, you need to manually install it. Follow these steps, and during the installation process, select `lightdm` as the Display Manager:
```
sudo apt update
sudo apt install -y xserver-xorg-video-dummy x11vnc xfce4
sudo vim /etc/X11/xorg.conf
sudo reboot
```
Edit the Xorg configuration file:
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
Start the x11vnc service:

```shell
sudo x11vnc -display :0 -auth /var/lib/lightdm/.Xauthority &
```

You can now use a VNC client on your PC to remotely connect using the `$bm1684_ip:0` address.

To connect using VNC Viewer:

（1）Download and install [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) on your PC, based on your system.

（2）Open VNC Viewer and skip the login step.

![](../../../bm1684_img/AIO-1684JD4/vnc-01.png)

（3）Enter the `$bm1684_ip:0` address of your AIO-1684JD4:

![](../../../bm1684_img/AIO-1684JD4/ubuntu-vnc-02.png)

（4）Click on the circular button next to the `linaro` username, select `xfce`, and enter the username and password (both are `linaro`):

![](../../../bm1684_img/AIO-1684JD4/ubuntu-vnc-03.png)

（5）You will successfully enter the xfce4 desktop environment.

![](../../../bm1684_img/AIO-1684JD4/ubuntu-vnc-04.png)
