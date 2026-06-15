# Display 使用

RK3576 拥有 3 路 vp (Video Port)，每一个 vp 端口都绑定了固定的显示控制器。详情可以查看设备树文件 rk3576.dtsi 中的 vop_out 节点。

比如 vp0 支持 dsi, hdmi/edp, dp0~2 的输出。

每一个 vpx 都有各自所能输出的最大分辨率：  
* vp0 最大可以输出 4K@120Hz   
* vp1 最大可以输出 2560x1600@60Hz  
* vp2 最大可以输出 1920x1080@60Hz    

软件上如果把 HDMI0 显示控制器连接在 vp0 上，且硬件 phy 支持 HDMI2.1，则 HDMI0 支持 4K@120Hz 的输出。

如果给每一个 vpx 都单独分配一个显示控制器的话，便可支持同一时间的三屏同显（异显）。

## ROC-RK3576-PC 显示接口的配置  

ROC-RK3576-PC 有三种显示输出接口，分别是 HDMI、Display Port 以及 MIPI DSI，可以做到多屏同显/异显。

下面对各个显示输出接口的配置和使用作基本的介绍，详细内容可以参考文件：   
* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-3576q.dtsi`  
* `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-rk3576q-mipi101-BSD1218-A101KL68.dtsi`  



### HDMI

ROC-RK3576-PC 硬件上有一个 HDMI 显示输出接口：

* HDMI 支持 HDMI2.1 协议，分辨率最高可以支持 4k@120Hz

#### 软件配置

在设备树中添加：

```
//打开 hdmi 功能
&hdmi {
	status = "okay";
	enable-gpios = <&gpio2 RK_PB1 GPIO_ACTIVE_HIGH>;
	rockchip,sda-falling-delay-ns = <360>;
};

//把 hdmi 的显示接口连接在 Port0
&hdmi_in_vp0 {
	status = "okay";
};

//打开 hdmi 音频输出
&hdmi_sound {
        status = "okay";
};

//打开 hdmi 的 硬件 phy
&hdptxphy_hdmi {
	status = "okay";
};


//打开 hdmi 的 开机 logo
&route_hdmi {
	status = "okay";
	connect = <&vp0_out_hdmi>;
};

```

### Display Port

ROC-RK3576-PC 有一个 Display Port 显示输出接口，支持  DP TX 1.4a 协议，分辨率最高可以支持 4k@120Hz。

* DP 接口

![](../../../rk3576_img/EC-R3576PC-FD/usage_display_dp_interface.png)


#### 软件配置

Display Port 软件上表示为 `dp0`，在设备树上添加：

```
// 打开 dp0 的音频输出功能
&spdif_tx3 {
	status = "okay";
};

&dp0_sound{
        status = "okay";
};

//打开 dp0 功能
&dp {
	status = "okay";
};

&dp0 {
        status = "okay";
};

//把 dp0 的显示接口连接在 port2
&dp0_in_vp2 {
        status = "okay";
};

/* 注意：目前 dp0 暂不支持 开机 logo 的显示功能 */

```

这里要注意，SDK 默认的软件配置是把 dp0 连接在 vp2 上面，这样会导致 dp0 最高只能输出 1920x1080@60Hz, 所以如果需要 4k@120Hz 的功能，需要把 dp0 连接在 vp0上面。


### MIPI DSI

ROC-RK3576-PC 有一路 MIPI DSI 显示输出接口，支持 DPHY2.0 和 4 Lane 的数据输出。  

#### 软件配置

固件默认适配的屏幕是 [Firefly V3S 版本屏幕](https://wiki.t-firefly.com/DM-M10R800-V3S/dm-m10r800-v3s.html)，


* DSI 接口

![](../../../rk3576_img/EC-R3576PC-FD/usage_display_mipi_v2_interface.png)
  
参考设备树：
```
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-aio-rk3576q-mipi101-BSD1218-A101KL68.dtsi
```

## ROC-RK3576-PC 多屏场景的配置

下面列出一些常规的场景下，关于 Portx 以及 显示控制器之间的配置方法：  

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

## 调试手段

* 在系统中设置分辨率

系统鼠标点击：‘设置->显示->HDMI->分辨率设置’  

* 获取 HDMI/Display Port 的 edid(以 HDMI 为例)
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

* 获取 HDMI/Display Port 所支持的分辨率(以 HDMI 为例)
```
:/ # cat /sys/class/drm/card0-HDMI-A-1/modes

```  
* 获取 HDMI/Display Port 的连接状态(以 HDMI 为例)
```
:/ # cat /sys/class/drm/card0-HDMI-A-1/status
connected
```  

* 获取系统中正在使用的 Video Portx（与所连接的显示控制器） 信息
```
:/ # cat /d/dri/0/summary
Video Port0: ACTIVE
    Connector:HDMI-A-1	Encoder: TMDS-403
	bus_format[2025]: YUV8_1X24
	overlay_mode[1] output_mode[f] SDR[0] color-encoding[BT.709] color-range[Limited]
    Display mode: 1920x1080p60
	clk[148500] real_clk[148500] type[48] flag[9]
	H: 1920 2008 2052 2200
	V: 1080 1084 1089 1125
    Cluster0-win0: ACTIVE
	win_id: 4
	format: AB24 little-endian (0x34324241)[AFBC] pixel_blend_mode[0] glb_alpha[0xff]
	color: SDR[0] color-encoding[BT.709] color-range[Full]
	rotate: xmirror: 0 ymirror: 0 rotate_90: 0 rotate_270: 0
	csc: y2r[0] r2y[1] csc mode[1]
	zpos: 0
	src: pos[0, 0] rect[1920 x 1080]
	dst: pos[0, 0] rect[1920 x 1080]
	buf[0]: addr: 0x0000000002e73000 pitch: 7680 offset: 0
Video Port2: DISABLED

```


一般如果遇到 HDMI/Display Port 无法显示的问题，都需要先执行上面的命令去看一下连接状态、edid 和分辨率是否正确。