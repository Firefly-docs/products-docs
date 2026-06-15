# Display



RK3576 has three video output ports, each video output port is bound to a fixed display controller, such as Port0 can be used to connect with display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1, other Portx and so on.

Each Portx has its own maximum resolution:
* Port0 can output up to 4K@120Hz
* Port1 can output up to 2560x1600@60Hz
* Port2 can output up to 1920x1080@60Hz

In software, if the HDMI0 display controller is connected to Port0, and the hardware phy supports HDMI2.1, then HDMI0 supports 4K@120Hz output.

If each Portx is assigned a separate display controller, it can support three-screen simultaneous display (different display) at the same time.

## AIO-3576JQ Displays the configuration of the interface

 AIO-3576JQ  There are three display output interfaces, namely HDMI, Display Port , which can achieve multi-screen simultaneous display/exclusive display. The interface diagram is as follows:

* HDMI/ Display Port/ MIPI DSI 

![](../../../rk3576_img/iCore-3576JQ/usage_display_interface.png)


The following is a basic introduction to the configuration and use of each display output interface. For details, please refer to the file:
* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi`  


### HDMI

AIO-3576JQ There are one HDMI display output interfaces on the hardware:

* HDMI supports HDMI2.1 protocol, resolution can support up to 4k@120Hz


#### Software configuration

The following takes the configuration of HDMI0 as an example.

```
//enable hdmi0
&hdmi {
	status = "okay";
	enable-gpios = <&gpio2 RK_PB0 GPIO_ACTIVE_HIGH>;
	rockchip,sda-falling-delay-ns = <360>;
};


//connect hdmi0 with Port0
&hdmi_in_vp0 {
	status = "okay";
};

//enable hdmi0 audio output
&hdmi_sound {
        status = "okay";
};

//enable hdmi0's phy
&hdptxphy_hdmi {
	status = "okay";
};

//enable hdmi0 logo of startup
&route_hdmi {
	status = "okay";
	connect = <&vp0_out_hdmi>;
};

```

### Display Port

AIO-3576JQ has a Display Port display output interface, supports DP TX 1.4a protocol, and resolution can support up to 4k@120Hz.


#### Software configuration

Display Port is represented as  `dp0` on the system status tree, add:

```
// enable dp0 audio output
&spdif_tx3 {
	status = "okay";
};

&dp0_sound{
        status = "okay";
};

//enable dp0
&dp {
	status = "okay";
};

&dp0 {
        status = "okay";
};


//connect dp0 with port2
&dp0_in_vp2 {
        status = "okay";
};

/* Note：Currently dp0 does not support the startup logo display function */

```

It should be noted here that SDK default software configuration is to connect dp0 to vp2, which will cause dp0 to output only 1920x1080@60Hz at most, so if you need 4k@120Hz functions, you need to connect dp0 to vp0. The software is modified as follows:

```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi b/kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi
index fc08eb7543..fee53ed2e9 100644
--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi
+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi
@@ -301,7 +301,7 @@ &dp0 {
        status = "okay";
 };

-&dp0_in_vp2 {
+&dp0_in_vp0 {
        status = "okay";
 };
```

## AIO-3576JQ  Multiscreen scene configuration

Here are some general scenarios about Portx and configuration methods between display controllers:

* HDMI（4K@120Hz） + Display Port
```
&hdmi_in_vp0 {
        status = "okay";
};

&dp0_in_vp2 {
        status = "okay";
};
```

* HDMI（1080P@60Hz） + Display Port (4K@60Hz)
```
&hdmi_in_vp2 { 
        status = "okay";
};      

&dp0_in_vp1 {
        status = "okay";
};
```

## Debug method

* Set resolution in the system

System mouse click: 'Settings — > Display — > HDMI- > resolution settings'

* Get the edid of HDMI/Display Port (take HDMI as an example)
```
:/ # busybox hexdump /sys/class/drm/card0-HDMI-A-1/edid
0000000 ff00 ffff ffff 00ff 040d 0030 0001 0000
0000010 1e01 0301 8b80 784e 502a a31f 4959 2497
0000020 4fbb 2153 0008 8081 c081 0081 c0d1 7c61
0000030 fc81 0101 0101 7404 7c00 70f6 805a 58fc
0000040 008a 1072 0053 1e00 3a02 1880 3871 402d
0000050 2c58 0045 1072 0053 1e00 0000 fc00 4300
0000060 5348 5648 200a 2020 2020 2020 0000 fd00
0000070 1700 0f4c 1e50 0a00 2020 2020 2020 4a01
0000080 0302 f07c 015f 0302 0504 0706 1190 1312
0000090 1514 1f16 2220 5e5d 605f 6261 6564 c266
00000a0 c4c3 c7c6 0932 0717 0715 5750 0106 0467
00000b0 3d03 c007 7e5f e601 4611 00d0 8070 4783
00000c0 0000 036e 000c 0020 3cb8 0020 0180 0302
00000d0 6d04 5dd8 01c4 8078 2267 0000 67cf e51f
00000e0 000f 3000 e363 0506 e301 c305 e201 ff00
00000f0 01eb d046 4800 42af 38a2 d727 0000 a400
0000100
```  

* Get the resolutions supported by HDMI/Display Port (take HDMI as an example)
```
:/# cat /sys/class/drm/card0-HDMI-A-1/modes
1920x1080
1920x1080i
1280x1024
1280x960
1280x720
1280x720
1280x720
720x480
720x480
720x480

```  
* Get the connection status of HDMI/Display Port (take HDMI as an example)
```
:/ # cat /sys/class/drm/card0-HDMI-A-1/status
connected
```  

* Get information on the Video Portx in use on the system (with the connected display controller)
```
:/# cat /sys/kernel/debug/dri/0/summary 
Video Port0: ACTIVE
    Connector:HDMI-A-1  Encoder: TMDS-179
        bus_format[2025]: YUV8_1X24
        overlay_mode[1] output_mode[f] SDR[0] color-encoding[BT.709] color-range[Limited]
    Display mode: 1920x1080p60
        dclk[148500 kHz] real_dclk[148500 kHz] aclk[500000 kHz] type[48] flag[5]
        H: 1920 2008 2052 2200
        V: 1080 1084 1089 1125
        Fixed H: 1920 2008 2052 2200
        Fixed V: 1080 1084 1089 1125
    Esmart0-win0: ACTIVE
        win_id: 0
        format: XR24 little-endian (0x34325258) pixel_blend_mode[0] glb_alpha[0xff]
        color: SDR[0] color-encoding[BT.601] color-range[Limited]
        rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
        csc: y2r[0] r2y[1] csc mode[1]
        zpos: 0
        src: pos[0, 0] rect[1920 x 1080]
        dst: pos[0, 0] rect[1920 x 1080]
        buf[0]: addr: 0x0000000000000000 pitch: 7680 offset: 0
Video Port2: DISABLED
```


Generally, if you encounter the problem that HDMI/Display Port cannot be displayed, you need to execute the above command to see if the connection status, edid and resolution are correct.