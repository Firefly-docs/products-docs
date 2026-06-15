# Display


![](../../../rk3588_img/common/usage_display_rk3588_vop.png)


RK3588 has four video output ports, each video output port is bound to a fixed display controller, such as Port0 can be used to connect with display controllers such as DP0, DP1, HDMI/eDP0 and HDMI/eDP1, other Portx and so on.

Each Portx has its own maximum resolution:
* Port0 can output up to 7680x4320@60Hz
* Port1 can output up to 4096x2304@60Hz
* Port2 can output up to 4096x2304@60Hz
* Port3 can output up to 1920x1080@60Hz

In software, if the HDMI0 display controller is connected to Port0, and the hardware phy supports HDMI2.1, then HDMI0 supports 8K@60Hz output.

If each Portx is assigned a separate display controller, it can support four-screen simultaneous display (different display) at the same time.

But from the software point of view, there are the following configuration considerations:

1. When RK3588 does 8K output, in fact, the chip uses the resources of Port0 and Port1 at the same time, but only borrows the Port0 port for output, so when the display controller connected to Port0 is connected to an external 8K display for output and the  Port1 is using. The display controller will work abnormally.

   For example, when HDMI0 (connected to Port0) is connected to 8K output, the display of DP0 (connected to Port1) will be abnormal. Only when the output of Port0 is less than or equal to 4K@60Hz, Port1 can output 4K@60Hz normally.
  


## Core-3588L  the configuration of the Display interface  

 Core-3588L  supports only two display output interfaces by default (but the 164-pin interface can be extended to VGA, Display Port1 and DSI display interfaces), namely HDMI and Display Port, which can be used for multi-screen display/different display, as shown in the interface diagram below.  

* HDMI0/ HDMI1/ Display Port

![](../../../rk3588_img/Core-3588L/usage_display_interface.jpg)   


The following is a basic introduction to the configuration and use of each display output interface. For details, please refer to the file:
* `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi`    



### HDMI

The Core-3588L hardware has two HDMI display output connectors.

* HDMI0 supports HDMI2.1 protocol and can support resolutions up to 7680x4320@60Hz
* HDMI1 supports HDMI2.0 protocol and can support resolutions up to 4096x2304@60Hz


#### Software Configuration

Since HDMI0 and HDMI1 are similar in software configuration, the following is an example of HDMI0 configuration, HDMI0 is represented as ``hdmi0`` in the software, add in the device tree.

```
/* hdmi0 */
// Turn on the hdmi0 function
&hdmi0 {
	enable-gpios = <&gpio4 RK_PA0 GPIO_ACTIVE_HIGH>;
	status = "okay";
};
//Connect the display interface of hdmi0 to Port0
&hdmi0_in_vp0 {
	status = "okay";
};
//turn on hdmi0 audio output
&hdmi0_sound {
	status = "okay";
};
//turn on hardware phy for hdmi0
&hdptxphy_hdmi0 {
	status = "okay";
};
//turn on hdmi0's boot logo
&route_hdmi0{
	status = "okay";
};

```

Note that since HDMI0 will be connected to Port0 by default here, i.e. in order to support 8K output. If HDMI1 is connected to Port1 at this time, e.g.

```
&hdmi1_in_vp1 {
        status = "okay";
};
```

then there will be an abnormal HDMI1 display as mentioned before, so to support HDMI0 (8K) + HDMI1 (4K) scenario, you need to connect HDMI1 to Port2, e.g.

```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc. dtsi
index afb176b9f8..099cccbf65 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi
@@ -94,7 +94,7 @@ &hdmi1 {
 &hdmi1_sound {
        status = "okay";
 };
-&hdmi1_in_vp1 {
+&hdmi1_in_vp2 {
         status = "okay";
 };
 &hdptxphy_hdmi1 {

```

### Display Port

Core-3588L has a Display Port display output interface, supports DP TX 1.4a protocol, and can support resolutions up to 7680x4320@30Hz.

#### software configuration

Display Port is represented as ``dp0`` in the software, add to the device tree.

```
// Turn on the audio output function of dp0
&spdif_tx2{
        status = "okay";
};

&dp0_sound{
        status = "okay";
};

//open dp0 function
&dp0 {
        status = "okay";
};

//connect dp0's display interface to port2
&dp0_in_vp2 {
        status = "okay";
};

//enable the PD power chip corresponding to the Type-C port
&usbc0{
        status = "okay";
    	interrupt-parent = <&gpio0>;
		interrupts = <RK_PD3 IRQ_TYPE_LEVEL_LOW>;
};

&vbus5v0_typec_pwr_en{
		status = "okay";
		gpio = <&gpio4 RK_PA3 GPIO_ACTIVE_HIGH>; 
};

/* Note: Currently dp0 does not support the boot logo display function */

```

Note here that the default software configuration of the SDK is to connect dp0 to vp2, which will cause dp0 to output up to 4096x2304@60Hz, so if you need the 7680x4320@30Hz function, you need to connect dp0 to vp0, and the software modification is as follows.

```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc. dtsi
index fc08eb7543..fee53ed2e9 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588-pc.dtsi
@@ -301,7 +301,7 @@ &dp0 {
        status = "okay";
 };

-&dp0_in_vp2 {
+&dp0_in_vp0 {
        status = "okay";
 };

```

## Core-3588L  Multiscreen scene configuration

Since Core-3588L only supports 3 display outputs by default, HDMI0/HDMI1/Display Port0, there are conventional scenarios configured as follows.

* Triple display (HDMI0 + HDMI1 + Display Port, at this time HDMI0 cannot access screens with a resolution greater than 4096x2304@60Hz, otherwise HDMI1's display will have an exception)

```
&hdmi0_in_vp0 {
        status = "okay";
};

&hdmi1_in_vp1 {
        status = "okay";
};

&dp0_in_vp2 {
        status = "okay";
};
```  

* Dual HDMI support 8K@60Hz + 4K@60Hz output

```
&hdmi0_in_vp0 {
        status = "okay";
};

&hdmi1_in_vp2 {
        status = "okay";
};
```
If other display interfaces (VGA, MIPI DSI and Display Port1) are extended from the 164 Pin interface, then the corresponding configuration scenarios can be referred to: [ITX-3588J Multi-screen Scenario Configuration](https://wiki.t-firefly.com/en/Core-3588J/usage_display.html#itx-3588j-displays-the-configuration-of-the-interface)


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