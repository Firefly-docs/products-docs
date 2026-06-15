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
  
2. At present, a Portx can only output one resolution format at the same time, so if the software configuration has multiple display interfaces with different resolutions connected to the same Portx, only one of them can be used at the same time. For example, VGA and DP can only use one of them at the same time (** Later with the update of the SDK, it can support the simultaneous connection of multiple display peripherals under the same Portx. **).



## AIO-3588L Displays the configuration of the interface

 AIO-3588L have a HDMI, interface diagram is as follows:

* HDMI
![](../../../rk3588_img/EC-A3588L/usage_display_dsi_interface.jpg)  


The following is a basic introduction to the configuration and use of each display output interface. For details, please refer to the file:
* `kernel/arch/arm64/boot/dts/rockchip/aio-3588jd4.dtsi`  


### HDMI

AIO-3588L There are two HDMI display output interfaces on the hardware:

* HDMI0 supports HDMI2.1 protocol, resolution can support up to 7680x4320@60Hz


#### Software configuration

The following takes the configuration of HDMI0 as an example.

```
//enable hdmi0
&hdmi0 {
        enable-gpios = <&gpio4 RK_PA0 GPIO_ACTIVE_HIGH>;
        status = "okay";
};

//connect hdmi0 with Port0
&hdmi0_in_vp0 {
        status = "okay";
};

//enable hdmi0 audio output
&hdmi0_sound {
        status = "okay";
};

//enable hdmi0's phy
&hdptxphy_hdmi0 {
        status = "okay";
};

//enable hdmi0 logo of startup
&route_hdmi0{
        status = "okay";
};

```

## AIO-3588L  Multiscreen scene configuration

Here are some general scenarios about Portx and configuration methods between display controllers:

* HDMI0（8K@60Hz） + Display Port（4K@60Hz） + MIPI DSI0
```
&hdmi0_in_vp0 {
        status = "okay";
};

&dsi0_in_vp3 {
        status = "okay";
};

&dp0_in_vp2 {
        status = "okay";
};
```
  
* HDMI0（8K@60Hz） + Display Port（4K@60Hz） + MIPI DSI1
```
&hdmi0_in_vp0 {
        status = "okay";
};

&dsi1_in_vp3 {
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