# Display 使用


![](../../../rk3588_img/common/usage_display_rk3588_vop.png)


RK3588S 拥有四路 Video 输出端口，每一个 Video 输出端口都绑定了固定的显示控制器，如 Port0 可以用于与 DP0、DP1、HDMI/eDP0 和 HDMI/eDP1 等显示控制器的连接，其他 Portx 以此类推。  

每一个 Portx 都有各自所能输出的最大分辨率：  
* Port0 最大可以输出 7680x4320@60Hz  
* Port1 最大可以输出 4096x2304@60Hz  
* Port2 最大可以输出 4096x2304@60Hz  
* Port3 最大可以输出 1920x1080@60Hz  

软件上如果把 HDMI0 显示控制器连接在 Port0 上，且硬件 phy 支持 HDMI2.1，则 HDMI0 支持 8K@60Hz 的输出。  

如果给每一个 Portx 都单独分配一个显示控制器的话，便可支持同一时间的四屏同显（异显）。  

但从软件的角度上，有以下的配置注意事项：  

1. RK3588S 在做 8K 输出的时候，其实芯片内部是同时使用了 Port0 以及 Port1 的资源，只是借用 Port0 口做输出，所以当与 Port0 连接的显示控制器连接外部的 8K 显示器做输出的时候，与 Port1 连接的显示控制器工作会有异常。  

   例如 HDMI0（接Port0) 接 8K 输出的时候，DP0（接Port1) 显示会有异常。只有当 Port0 输出小于等于4K@60Hz 的时候，Port1 才可以正常输出 4K@60Hz。  



## AIO-3588JQ  显示接口的配置  

AIO-3588JQ  有二种显示输出接口，分别是 HDMI、Display Port ，可以做到双屏同显/异显，接口图如下所示：  

* HDMI0/ Display Port

![](../../../rk3588_img/iCore-3588JQ/usage_display_interface.png)  


下面对各个显示输出接口的配置和使用作基本的介绍，详细内容可以参考文件：   
* `kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi`  

### HDMI

AIO-3588JQ  硬件上有一个 HDMI 显示输出接口：

* HDMI0 支持 HDMI2.1 协议，分辨率最高可以支持 7680x4320@60Hz


#### 软件配置

下面描述以 HDMI0 的配置为例，HDMI0 软件上表示为 `hdmi0`,在设备树中添加：

```
//打开 hdmi0 功能
&hdmi0 {
        enable-gpios = <&gpio4 RK_PB2 GPIO_ACTIVE_HIGH>;
        status = "okay";
};

//把 hdmi0 的显示接口连接在 Port0
&hdmi0_in_vp0 {
        status = "okay";
};

//打开 hdmi0 音频输出
&hdmi0_sound {
        status = "okay";
};

//打开 hdmi0 的 硬件 phy
&hdptxphy_hdmi0 {
        status = "okay";
};

//打开 hdmi0 的 开机 logo
&route_hdmi0{
        status = "okay";
};

```

需要注意的是，由于这里默认会把 HDMI0 连接在 Port0 上，即为了支持 8K 输出。如果这个时候 DP0 连接在了 Port1 端口，如：

```
&dp0_in_vp1 {
        status = "okay";
};
```

则会出现前文说的 DP0 显示异常的情况，所以如果要支持 HDMI0（8K）+ DP0（4K）场景，需要把 DP0 连接在 Port2 上，如：

```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi
index afb176b9f8..099cccbf65 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi
@@ -94,7 +94,7 @@ &dp0 {

+&dp0_in_vp2 {
         status = "okay";
 };
```



### Display Port

AIO-3588JQ  有一个 Display Port 显示输出接口，支持  DP TX 1.4a 协议，分辨率最高可以支持 7680x4320@30Hz。


#### 软件配置

Display Port 软件上表示为 `dp0`，在设备树上添加：

```
// 打开 dp0 的音频输出功能
&spdif_tx2{
        status = "okay";
};

&dp0_sound{
        status = "okay";
};

//打开 dp0 功能
&dp0 {
        status = "okay";
};

//把 dp0 的显示接口连接在 port2
&dp0_in_vp2 {
        status = "okay";
};

//使能对应 Type-C 口的 PD 电源芯片
&usbc0{
        status = "okay";
        interrupt-parent = <&gpio0>;
        interrupts = <RK_PC4 IRQ_TYPE_LEVEL_LOW>;
};

/* typec0 */
&vbus5v0_typec_pwr_en {
		status = "okay";
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		gpio = <&gpio1 RK_PB1 GPIO_ACTIVE_HIGH>;
		vin-supply = <&vcc5v0_usb>;
		pinctrl-names = "default";
		pinctrl-0 = <&typec5v_pwren>;
};

/* 注意：目前 dp0 暂不支持 开机 logo 的显示功能 */

```

这里要注意，SDK 默认的软件配置是把 dp0 连接在 vp2 上面，这样会导致 dp0 最高只能输出 4096x2304@60Hz，所以如果需要 7680x4320@30Hz 的功能，需要把 dp0 连接在 vp0上面，软件修改如下：

```
diff --git a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi
index fc08eb7543..fee53ed2e9 100644
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/roc-rk3588s-pc.dtsi
@@ -301,7 +301,7 @@ &dp0 {
        status = "okay";
 };

-&dp0_in_vp2 {
+&dp0_in_vp0 {
        status = "okay";
 };

```

## AIO-3588JQ  双屏场景的配置

下面列出一些常规的场景下，关于 Portx 以及 显示控制器之间的配置方法：  

* HDMI0（8K@60Hz） + Display Port（4K@60Hz）
```
&hdmi0_in_vp0 {
        status = "okay";
};

&dp0_in_vp2 {
        status = "okay";
};
```
  
* HDMI0（8K@60Hz） + Display Port（4K@60Hz） 
```
&hdmi0_in_vp0 {
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
* 获取 HDMI/Display Port 的连接状态(以 HDMI 为例)
```
:/ # cat /sys/class/drm/card0-HDMI-A-1/status
connected
```  

* 获取系统中正在使用的 Video Portx（与所连接的显示控制器） 信息
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


一般如果遇到 HDMI/Display Port 无法显示的问题，都需要先执行上面的命令去看一下连接状态、edid 和分辨率是否正确。