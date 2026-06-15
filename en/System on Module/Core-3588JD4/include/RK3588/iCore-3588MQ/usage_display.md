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



## AIO-3588JD4 Displays the configuration of the interface

 AIO-3588JD4  There are five display output interfaces, namely HDMI, Display Port, VGA ,EDP and MIPI DSI, which can achieve multi-screen simultaneous display/exclusive display. The interface diagram is as follows:

* HDMI0/ HDMI1/ Display Port/ VGA
![](../../../rk3588_img/Core-3588JD4/usage_display_interface.jpg)  

* MIPI DSI0/ MIPI DSI1 / EDP
![](../../../rk3588_img/Core-3588JD4/usage_display_dsi_interface.jpg) 


The following is a basic introduction to the configuration and use of each display output interface. For details, please refer to the file:
* `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi`  
* `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q-mipi101-M101014-BE45-A1.dts`
* `kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q-edp-NV156FHM-T06.dts`



### HDMI

AIO-3588JD4 There are two HDMI display output interfaces on the hardware:

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

It should be noted that since HDMI0 will be connected to Port0 by default, that is, in order to support 8K output. If DP0 is connected to the Port1 port at this time, such as:

```
&dp0_in_vp1 {
        status = "okay";
};
```

There will be an abnormal display of DP0 as mentioned above, so if you want to support HDMI0 (8K) + DP0 (4K) scenarios, you need to connect DP0 to Port2, such as:

```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi
index afb176b9f8..099cccbf65 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi
@@ -94,7 +94,7 @@ &dp0 {

-&dp0_in_vp1 {
+&dp0_in_vp2 {
         status = "okay";
 };
```



### Display Port

AIO-3588JD4 has a Display Port display output interface, supports DP TX 1.4a protocol, and resolution can support up to 7680x4320@30Hz.


#### Software configuration

Display Port is represented as  `dp0` on the system status tree, add:

```
// enable dp0 audio output
&spdif_tx2{
        status = "okay";
};

&dp0_sound{
        status = "okay";
};

//enable dp0
&dp0 {
        status = "okay";
};

//connect dp0 with port2
&dp0_in_vp2 {
        status = "okay";
};

//enable Type-C's PD power chip
&usbc0{
        status = "okay";
        interrupt-parent = <&gpio0>;
        interrupts = <RK_PD3 IRQ_TYPE_LEVEL_LOW>;
};

&vbus5v0_typec_pwr_en{
        status = "okay";
        gpio = <&pca9555 PCA_IO1_4 GPIO_ACTIVE_HIGH>;  //PCA_IO 14
};

/* Note：Currently dp0 does not support the startup logo display function */

```

It should be noted here that SDK default software configuration is to connect dp0 to vp2, which will cause dp0 to output only 4096x2304@60Hz at most, so if you need 7680x4320@30Hz functions, you need to connect dp0 to vp0. The software is modified as follows:

```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi
index 885e6773b80..de6ed6d5e7c 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q.dtsi
@@ -167,7 +167,7 @@ &dp0 {
        status = "okay";
 };
 
-&dp0_in_vp2 {
+&dp0_in_vp0 {
        status = "okay";
 };
 
@@ -177,7 +177,7 @@ &hdmi0 {
        status = "okay";
 };
 
-&hdmi0_in_vp0 {
+&hdmi0_in_vp2 {
        status = "okay";
 };
```

### VGA

AIO-3588JD4 has a VGA display output interface, which is implemented by DP to VGA chip. Limited by the VGA protocol, the resolution can support up to 1080p@60Hz.

#### Software configuration

Since it is DP to VGA, the software configuration is still configured like Display Port, just need to add a hot-swap detection pin, which is represented by the software as  `dp1`, add it to the system status tree:


```
//connect dp1（VGA) with port2
&dp1_in_vp2 {
        status = "okay";
};

// enable dp1（VGA) ，config the hotplug pin
&dp1 {
    pinctrl-names = "default";
    pinctrl-0 = <&dp1_hpd>;
    hpd-gpios = <&gpio1 RK_PA4 GPIO_ACTIVE_HIGH>;
    status = "okay";
};

//enable dp1（VGA） logo of startup
&route_dp1{
    status = "okay";
    connect = <&vp2_out_dp1>;
};
```


### MIPI DSI

AIO-3588JD4 There are two MIPI DSI display output interfaces, both support DPHY2.0 and 4 Lane data output, and can output up to 4096x2304@60Hz (depending on the connected Portx).

#### Software configuration

Since DSI0 and DSI1 are similar in software configuration, here the configuration of DSI0 is taken as an example. The external screen is  [Firefly V2 Version](https://wiki.t-firefly.com/en/DM-M10R800-V2/dm-m10r800-v2.html), and the DSI0 software is represented as `dsi0`.

Combining AIO-3588JD4  DSI0 interface and screen timing

* DSI0 interface
![](../../../rk3588_img/Core-3588JD4/usage_display_mipi_v2_interface.png)
  

* V2 screen display timing
![](../../../rk3588_img/common/usage_display_mipi_v2_timing.jpg)
  
  

* V2 screen power-on timing
![](../../../rk3588_img/common/usage_display_mipi_v2_power_on.png) 
  
  

* V2 screen power-down timing
![](../../../rk3588_img/common/usage_display_mipi_v2_power_off.png)
  
  

* V2 screen power-up symbol reference
![](../../../rk3588_img/common/usage_display_mipi_v2_power_menu.png) 
  


Add to the system status tree:

```
//config the backlight
backlight: backlight {
        ...
        status = "okay";
        compatible = "pwm-backlight";
        enable-gpios = <&gpio2 RK_PB5 GPIO_ACTIVE_LOW>;
        pwms = <&pwm12 0 50000 1>;

        ...
}
&pwm12 {
        pinctrl-0 = <&pwm12m1_pins>;
        status = "okay";
};

//enable the  dsi0 logo og startup
&route_dsi0 {
        status = "okay";
        connect = <&vp3_out_dsi0>;
};

//connect dsi0 with Port3 
&dsi0_in_vp3 {
        status = "okay";
};

//enable DPHY
&mipi_dcphy0 {
        status = "okay";
};

//enable dsi0（Note:this part is more important which contants the timing of the display）
&dsi0 {
	status = "okay";
	//rockchip,lane-rate = <1000>;
	dsi0_panel: panel@0 {
		status = "okay";
		compatible = "simple-panel-dsi";
		reg = <0>;

		//select the backlight node of the previous configuration
		backlight = <&backlight>; 
		
		//sets the LCD enable pin
		enable-gpios = <&pca9555 PCA_IO0_2 GPIO_ACTIVE_HIGH>;

		//sets the LCD reset pin
		reset-gpios = <&gpio2 RK_PB4 GPIO_ACTIVE_LOW>; 

		//sets the timing of startup of LCD 
		enable-delay-ms = <50>;
        prepare-delay-ms = <200>;
        reset-delay-ms = <50>;
        init-delay-ms = <55>;
        unprepare-delay-ms = <50>;
        disable-delay-ms = <20>;
        mipi-data-delay-ms = <200>;
        size,width = <120>;
        size,height = <170>;

		//set the output mode of DSI (DPHY). the default mode is Video  
		dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;

		//sets the output format for DSI pixel data, depending on whether the receiving screen supports the format  
		dsi,format = <MIPI_DSI_FMT_RGB888>;

		//set the number of lanes to be used. Default is 4 lanes
		dsi,lanes  = <4>;

		//set the power-on command for the MIPI DSI
		panel-init-sequence = [
			//39 00 04 B9 83 10 2E
			// 15 00 02 CF FF
			05 78 01 11
			05 32 01 29
			//15 00 02 35 00
		];

		//set the power-on command for the MIPI DSI
		panel-init-sequence = [
            05 78 01 11
            05 32 01 29
        ];

		//set the power-off command for the MIPI DSI
        panel-exit-sequence = [
            05 00 01 28
            05 00 01 10
        ];

		//set the display timing of the LCD screen (this part can be obtained from the corresponding screen, as shown in the figure above)  
		disp_timings0: display-timings {
			native-mode = <&dsi0_timing0>;
			dsi0_timing0: timing0 {

				/* the conversion formula for display timing sequence is generally：
				（hactive + hsync-len + hback-porch + hfront-porch）   
							x
				（ vactive + vsync-len + vback-porch + vfront-porch）x fps
				 = clock-frequency
				*/
				clock-frequency = <72600000>;//<80000000>;
				hactive = <800>;//<768>;
				vactive = <1280>;
				hsync-len = <14>;   //20, 50,10
				hback-porch = <26>; //50, 56,10
				hfront-porch = <32>;//50, 30,180
				vsync-len = <8>;//4
				vback-porch = <20>;//4
				vfront-porch = <80>;//8
				hsync-active = <0>;
				vsync-active = <0>;
				de-active = <0>;
				pixelclk-active = <0>;
			};
		};

		ports {
			#address-cells = <1>;
			#size-cells = <0>;

			port@0 {
				reg = <0>;
				panel_in_dsi: endpoint {
					remote-endpoint = <&dsi_out_panel>;
				};
			};
		};
	};

	ports {
		#address-cells = <1>;
		#size-cells = <0>;

		port@1 {
			reg = <1>;
			dsi_out_panel: endpoint {
			remote-endpoint = <&panel_in_dsi>;
			};
		};
	};
};

//enable the touch function on the screen
&i2c1{
	status = "okay";
	pinctrl-names = "default";
	pinctrl-0 = <&i2c1m2_xfer>;

	hxchipset@48{
		status = "okay";
		compatible = "himax,hxcommon";
		reg = <0x48>;
		//set the reset pin for the touch function
		himax,rst-gpio =  <&gpio3 RK_PC1 GPIO_ACTIVE_HIGH>;

		//sets the interrupt pin for the touch function
		himax,irq-gpio = <&gpio3 RK_PC0 IRQ_TYPE_LEVEL_HIGH>;

		pinctrl-names = "default";
        	pinctrl-0 = <&touch_int0>;
		//pinctrl-names = "default";
		//pinctrl-0 = <&lcd_tp_int>;

		himax,panel-coords = <0 800 0 1280>;      //Range of touch
		himax,display-coords = <0 800 0 1280>;    //resolution
		report_type = <1>;
	};
};

```
				
When configuring MIPI DSI, if there are abnormal phenomena, such as black screen, picture stretching, display noise, etc., you need to pay attention to troubleshooting:

1. Shows whether the timing configuration is correct, especially the DCLK configuration.

2. Is the power-on timing correct, in the file  `kernel-5.10/drivers/gpu/drm/panel/panel-simple.c`
In the`panel_simple_prepare` and `panel_simple_unprepare` functions, the power-up timing and gpio port configured in the system status tree are called.

   If you choose to enable the boot logo during the uboot phase, you also need to troubleshoot the `u-boot/drivers/video/drm/rockchip_panel.c`  and `panel_simple_prepare` functions in the `panel_simple_unprepare` file.

3. Take an oscilloscope to see if the power-on timing is correct, mainly to confirm whether the timing between the LCD enable pin, the reset pin and the on-screen power-on command is correct.

### EDP

AIO-3588JD4 There are one EDP display output interfaces,and can output up to 4096x2304@60Hz (depending on the connected Portx).

#### Software configuration

The EDP DTS configuration in the DTS source file:
`kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-aio-3588q-edp_NV156FHM-T06.dts`
```
edp_panel: edp-panel {
        compatible = "simple-panel";
        status = "okay";

        //select the backlight node of the previous configuration
        backlight = <&backlight>;
        
        //sets the Power enable pin
        pwr-gpios = <&pca9555_1 PCA_IO0_1 GPIO_ACTIVE_HIGH>;
        pwr-delay-ms = <200>;

        //sets the EDP enable pin
        enable-gpios = <&pca9555_1 PCA_IO0_2 GPIO_ACTIVE_HIGH>; // EDP_BL_EN

        //set the display timing of the EDP screen
        display-timings {
            native-mode = <&timing0>;
            timing0: timing0 {
                clock-frequency = <140000000>;
                hactive = <1920>;
                vactive = <1080>;
                hfront-porch = <40>;
                hsync-len = <40>;
                hback-porch = <80>;
                vfront-porch = <16>;
                vsync-len = <8>;
                vback-porch = <16>;
                hsync-active = <0>;
                vsync-active = <0>;
                de-active = <0>;
                pixelclk-active = <0>;
            };
        };

        ports {
            panel_in_edp1: endpoint {
                remote-endpoint = <&edp1_out_panel>;
            };
        };
    };

    //config the backlight
    backlight: backlight {
        status = "okay";
        compatible = "pwm-backlight";
        pwms = <&pwm8 0 500000 0>;

        ...
};

&pwm8 {
    pinctrl-0 = <&pwm8m2_pins>;
    status = "okay";
};


&edp1 {
    //force-hpd;
    hpd-gpios = <&gpio3 RK_PD5 GPIO_ACTIVE_HIGH>;
    status = "okay";

    ports {
        port@1 {
            reg = <1>;

            edp1_out_panel: endpoint {
                remote-endpoint = <&panel_in_edp1>;
            };
        };
    };
};

//enable the  edp1 logo og startup
&route_edp1 {
    status = "okay";
    connect = <&vp2_out_edp1>;
};

//connect edp1 with Port2 
&edp1_in_vp2 {
    status = "okay";
};

&hdptxphy1 {
    status = "okay";
};

&pca9555_1 {
    interrupt-controller;
    interrupt-parent = <&gpio1>; 
    interrupts = <RK_PC6 IRQ_TYPE_LEVEL_HIGH>;
    suspend-io-state = <0xFFF3>;
    resume-io-state  = <0xFFFF>;
    pinctrl-names = "default";
    pinctrl-0 = <&pca9555_int1>;

};

//enable the touch function on the screen
&i2c1{
    status = "okay";

    goodix_ts@5d {
        status = "okay";
        compatible = "goodix,gt9xxx";
        reg = <0x5d>;
        interrupt-parent = <&pca9555_1>;
        interrupts = <0 17 0x2008 0>;

        //sets the Power enable pin
        goodix,pwr-gpio = <&pca9555_1 PCA_IO0_0 GPIO_ACTIVE_HIGH>;

        //set the reset pin for the touch function
        goodix,rst-gpio = <&pca9555_1 PCA_IO1_7 GPIO_ACTIVE_HIGH>;

        //sets the interrupt pin for the touch function
        goodix,irq-gpio = <&gpio1 RK_PC6 IRQ_TYPE_LEVEL_LOW>;
        flip-x = <0>;
        flip-y = <0>;

        //read-tp-thread; //When this variable is defined, the edp screen reads tp data using the interrupt method, otherwise it uses the polling method

        ...
    };
};

&pinctrl {
    pca9555int {
        pca9555_int1: pca9555-int1 {
            rockchip,pins = <1 RK_PC6 RK_FUNC_GPIO &pcfg_pull_up>;
        };
    };
```

## AIO-3588JD4  Multiscreen scene configuration

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