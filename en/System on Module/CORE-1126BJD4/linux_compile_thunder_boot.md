## Compile Thunder-boot firmware
Currently, the latest SDK of CORE-1126-JD4 has been adapted to Quickboot. Compile Quickboot firmware options:
```
# Compile CORE-1126-JD4 V1.1 Quickboot firmware The adapted screen is DM-M10R800 V2 screen module
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-jd4-BE-45-tb-v11.mk
```

After the Quickboot firmware is started, the MIPI DM-M10R800 V2 screen module can preview the camera screen. If connected to the network, you can also preview the camera push rtsp.
* Linux PC preview command:
```
vlc rtsp://<device ip address>
```
* Windows PC needs to download the vlc application and enter the streaming address to preview.

![](/img/CORE-1126BJD4/windows_vlc.jpg)
Note: This quick start firmware is only applicable to the CORE-1126-JD4 V1.1 version of the core board. For the core board version, please refer to the silk screen on the hardware.