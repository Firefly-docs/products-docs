# Display Usage

RK3576 has 3 vp (Video Port), every vp support some display interfaces. Check vop_out node in device tree rk3576.dtsi for details.

For example: vp0 supports dsi, hdmi/edp, dp0~2 display.

Max resolution@fps for each vp:
* vp0 Up to 4K@120Hz
* vp1 Up to 2560x1600@60Hz
* vp2 Up to 1920x1080@60Hz

If you bind HDMI0 to vp0, and your hardware supports HDMI2.1, then HDMI0 can output 4K120Hz.

Bind a screen to every vp then you can get three screens display at same time, with same or different contents.

## ROC-RK3576-PC Display Configuration
There are three display output interfaces, namely HDMI, Display Port, LVDS and MIPI DSI, which can achieve multi-screen simultaneous display/exclusive display.

The following is a basic introduction to the configuration and use of each display output interface. For details, please refer to the file:

* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts`  
* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dtsi`  
* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c-mipi101-BSD1218-A101KL68.dtsi`  
* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c-lvds10.1-DM-M10R800.dtsi`

### HDMI

ROC-RK3576-PC There are one HDMI display output interfaces on the hardware:

* HDMI supports HDMI2.1 protocol, resolution can support up to 4k@120Hz

* HDMI interface

![](../../../rk3576_img/EC-R3576PC-FD/usage_display_hdmi_interface.jpg)

#### Software configuration

The following takes the configuration of HDMI as an example.
```
//enable hdmi
&hdmi {
	status = "okay";
	enable-gpios = <&gpio4 RK_PC6 GPIO_ACTIVE_HIGH>;
};

//connect hdmi with Port0
&hdmi_in_vp0 {
	status = "okay";
};

//enable hdmi audio output
&hdmi_sound {
        status = "okay";
};

//enable hdmi's phy
&hdptxphy_hdmi {
	status = "okay";
};

//enable hdmi logo of startup
&route_hdmi {
	status = "okay";
	connect = <&vp0_out_hdmi>;
};

```

### Display Port

ROC-RK3576-PC has a Display Port display output interface, supports DP TX 1.4a protocol, and resolution can support up to 4k@120Hz.

* DP interface

![](../../../rk3576_img/EC-R3576PC-FD/usage_display_dp_interface.jpg)

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
@@ -301,7 +301,7 @@ &dp0 {
        status = "okay";
 };

-&dp0_in_vp2 {
+&dp0_in_vp0 {
        status = "okay";
 };
```
### MIPI DSI

ROC-RK3576-PC has a MIPI DSI display output interface, support DPHY2.0 and 4 Lane data output.

#### Software configuration

The firmware supports screen by default is [Firefly V3S Version](https://wiki.t-firefly.com/en/DM-M10R800-V3S/dm-m10r800-v3s.html)

* DSI interface

![](../../../rk3576_img/EC-R3576PC-FD/usage_display_mipi_v2_interface.jpg)
  
Please refer to the device-tree file:
```
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-rk3576q-mipi101-BSD1218-A101KL68.dtsi
```

### LVDS

ROC-RK3576-PC has a LVDS display output interface，and resolution can support up to 2K by TTL to LVDS .

#### Software configuration

The firmware supports screen by default is [Firefly DM-M10R800](https://wiki.t-firefly.com/en/DM-M10R800/dm-m10r800.html)，

* LVDS interface

![](../../../rk3576_img/EC-R3576PC-FD/usage_display_lvds_interface.jpg)

Please refer to the device-tree file:
```
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c-lvds10.1-DM-M10R800.dtsi
```

It should be noted here that SDK default software configuration is to connect LVDS to vp1, which will cause DSI cannot connect to vp1. 

The software is modified as follows:

```
--- a/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
+++ b/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
@@ -24,8 +24,8 @@
 #define SATA           0
 #define PCIE           1
 #define EDP            0
-#define DSI            1
-#define LVDS           0
+#define DSI            0
+#define LVDS           1
```

### eDP

ROC-RK3576-PC  Hardware reserve an eDP display output interface 40Pin / 30Pin (default hardware used as HDMI function) :

#### Software configuration

Please refer to the device-tree file:
```
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c-edp-NE156QUM-N66.dtsi
```

**It should be noted here that eDP and HDMI function is a multiplexing relationship, eDP can be selected as a 30Pin or 40Pin seat, no special requirements are the default HDMI version.**

The software is modified as follows:
```
--- a/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
+++ b/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576c.dts
@@ -23,7 +23,7 @@
 #define DP             1
 #define SATA           0
 #define PCIE           1
-#define EDP            0
+#define EDP            1
 #define DSI            1
 #define LVDS           0
```

## ROC-RK3576-PC  Multiscreen scene configuration

Here are some general scenarios about Portx and configuration methods between display controllers:

* HDMI（4K@120Hz） + Display Port + MIPI DSI (2K@60Hz）
```
&hdmi_in_vp0 {
        status = "okay";
};

&dsi_in_vp1 {
        status = "okay";
};

&dp0_in_vp2 {
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
:/ # cat /sys/class/drm/card0-HDMI-A-1/modes
3840x2160
7680x4320
7680x4320
7680x4320
7680x4320
7680x4320
7680x4320
7680x4320
7680x4320
4096x2160
4096x2160
4096x2160
4096x2160
4096x2160
4096x2160
4096x2160

```  
* Get the connection status of HDMI/Display Port (take HDMI as an example)
```
:/ # cat /sys/class/drm/card0-HDMI-A-1/status
connected
```  

* Get information on the Video Portx in use on the system (with the connected display controller)
```
:/ # cat /d/dri/0/summary
Video Port0: ACTIVE
    Connector: HDMI-A-1
        bus_format[2026]: UYYVYY8_0_5X24
        overlay_mode[1] output_mode[e] color_space[3], eotf:0
    Display mode: 7680x4320p60
        clk[2376000] real_clk[2376000] type[40] flag[5]
        H: 7680 8232 8408 9000
        V: 4320 4336 4356 4400
    Cluster0-win0: ACTIVE
        win_id: 0
        format: AB24 little-endian (0x34324241)[AFBC] SDR[0] color_space[0] glb_alpha[0xff]
        rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
        csc: y2r[0] r2y[1] csc mode[1]
        zpos: 0
        src: pos[0, 0] rect[3840 x 2160]
        dst: pos[0, 0] rect[7680 x 4320]
        buf[0]: addr: 0x0000000010971000 pitch: 15360 offset: 0
Video Port1: DISABLED
Video Port2: DISABLED
Video Port3: DISABLED
```


Generally, if you encounter the problem that HDMI/Display Port cannot be displayed, you need to execute the above command to see if the connection status, edid and resolution are correct.
