# Remote Desktop X11VNC

x11vnc is a tool that allows administrators to connect to the server's real X desktop directly through the VNC Viewer.
## Debian
RK182X开发套件 has installed the xfce4 desktop and configured the x11vnc related environment by default. The user directly executes the following command to wait for the connection:

```shell
sudo x11vnc -display :0 -auth /var/lib/lightdm/.Xauthority &
```

Then you can use `$bm1684_ip:0` address to VNC remote connection on the PC.

The following is the specific operation process of VNC Viewer:

(1) The user first downloads [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) from the official website, and selects the corresponding installation package according to the PC system to download and install.

(2) After the installation is complete, open VNC Viewer and skip login:

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/vnc-01.png)

(3) Enter `$bm1684_ip:0` address of RK182X开发套件:

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/vnc-02.png)

(4) Enter the user and password, both `linaro`:

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/vnc-03.png)

(5) Successfully entered the xfce4 desktop:

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/vnc-04.png)

