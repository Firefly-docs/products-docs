# Camera 使用


## MIPI CSI用法
RK3566/RK3568平台仅有一个标准物理mipi csi2 dphy,可以工作在两个模式: full mode 和split
mode, 拆分为csi2_dphy0/csi2_dphy1/csi2_dphy2三个逻辑dphy(参见rk3568.dtsi)

### Full Mode
* 仅使用csi2_dphy0,csi2_dphy0与csi2_dphy1/csi2_dphy2互斥,不可同时使用;
* data lane最大4 lanes;
* 最大速率2.5Gbps/lane;

### Split Mode
* 仅使用csi2_dphy1和csi2_dphy2, 与csi2_dphy0互斥,不可同时使用;
* csi2_dphy1和csi2_dphy2可同时使用;
* csi2_dphy1和csi2_dphy2各自的data lane最大是2 lanes;
* csi2_dphy1对应物理dphy的lane0/lane1;
* csi2_dphy2对应物理dphy的lane2/lane3;
* 最大速率2.5Gbps/lane

![](../../img/rk356x_mipi_csi_mode.png)

简单点来讲，如果用单目摄像头我们可以配置full mode，若使用双目摄像头我们可以配置split mode

## Full Mode配置
**链接关系**: sensor->csi2_dphy0->isp

### Full Mode设备树配置要点

### 配置sensor端
我们需要根据板子原理图的MIPI CSI接口找到sensor是挂在哪个I2C总线上，然后在对应的I2C节点配置camera节点，正确配置camera模组的I2C设备地址、引脚等属性。如下Core-3568J的xc7160配置：
```shell
&i2c4 {
        status = "okay";
        XC7160: XC7160b@1b{
                status = "okay";
                compatible = "firefly,xc7160";
                reg = <0x1b>;
                clocks = <&cru CLK_CIF_OUT>;
                clock-names = "xvclk";
                power-domains = <&power RK3568_PD_VI>;
                pinctrl-names = "default";
                pinctrl-0 = <&cif_clk>;
                power-gpios = <&pca9555 PCA_IO0_4 GPIO_ACTIVE_LOW>;
                reset-gpios = <&pca9555 PCA_IO0_0 GPIO_ACTIVE_HIGH>;
                pwdn-gpios = <&pca9555 PCA_IO0_1 GPIO_ACTIVE_HIGH>;
                firefly,clkout-enabled-index = <0>;
                rockchip,camera-module-index = <0>;
                rockchip,camera-module-facing = "back";
                rockchip,camera-module-name = "NC";
                rockchip,camera-module-lens-name = "NC";
                port {
                        xc7160_out: endpoint {
                                remote-endpoint = <&mipi_in_ucam4>;
                                data-lanes = <1 2 3 4>;
                        };
                };
        };
};

```

### csi2_dphy0相关配置
csi2_dphy0与csi2_dphy1/csi2_dphy2互斥,不可同时使用。另外需要使能csi2_dphy_hw节点
```shell
&csi2_dphy0 {
        status = "okay";
        /*
        * dphy0 only used for full mode,
        * full mode and split mode are mutually exclusive
        */
        ports {
                #address-cells = <1>;
                #size-cells = <0>;
                port@0 {
                        reg = <0>;
                        #address-cells = <1>;
                        #size-cells = <0>;
...
                        mipi_in_ucam4: endpoint@5 {
                                reg = <5>;
                                remote-endpoint = <&xc7160_out>;
                                data-lanes = <1 2 3 4>;
                        };
                };
                port@1 {
                        reg = <1>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        csidphy_out: endpoint@0 {
                                reg = <0>;
                                remote-endpoint = <&isp0_in>;
                        };
                };
        };
};

&csi2_dphy_hw {
   status = "okay";
};

&csi2_dphy1 {
        status = "disabled";
};

&csi2_dphy2 {
        status = "disabled";
};
```

### isp相关配置
其中rkisp_vir0节点的`remote-endpoint`指向`csidphy_out`
```shell
&rkisp {
   status = "okay";
};

&rkisp_mmu {
   status = "okay";
};

&rkisp_vir0 {
   status = "okay";
   port {
      #address-cells = <1>;
      #size-cells = <0>;

      isp0_in: endpoint@0 {
         reg = <0>;
         remote-endpoint = <&csidphy_out>;
      };
   };
};
```


## Split Mode配置
**链接关系**: 

sensor1->csi_dphy1->isp_vir0

sensor2->csi_dphy2->mipi_csi2->vicap->isp_vir1

### Split Mode设备树配置要点

### 配置sensor端
我们需要根据板子原理图的MIPI CSI接口找到两个sensor是挂在哪个I2C总线上，然后在对应的I2C节点配置两个camera节点，正确配置camera模组的I2C设备地址、引脚等属性。如下Core-3568J的gc2053/gc2093配置：
```shell
&i2c4 {
            status = "okay";
           gc2053: gc2053@37 { //IR
                status = "okay";
                compatible = "galaxycore,gc2053";
                reg = <0x37>;

                avdd-supply = <&vcc_camera>;
                power-domains = <&power RK3568_PD_VI>;
                clock-names = "xvclk";
                pinctrl-names = "default";
		clocks = <&pmucru CLK_WIFI>;
                pinctrl-0 = <&refclk_pins>;
                power-gpios = <&pca9555 PCA_IO0_0 GPIO_ACTIVE_HIGH>; //IR_PWR_EN
                pwdn-gpios = <&pca9555 PCA_IO0_4 GPIO_ACTIVE_LOW>;
                firefly,clkout-enabled-index = <1>;
                rockchip,camera-module-index = <0>;
                rockchip,camera-module-facing = "back";
                rockchip,camera-module-name = "YT-RV1109-2-V1";
                rockchip,camera-module-lens-name = "40IR-2MP-F20";
                port {
                        gc2053_out: endpoint {
                                        remote-endpoint = <&dphy1_in>;
                                        data-lanes = <1 2>;
                        };
                };
        };
        gc2093: gc2093b@7e{ //RGB
                status = "okay";
                compatible = "galaxycore,gc2093";
                reg = <0x7e>;

                avdd-supply = <&vcc_camera>;
                power-domains = <&power RK3568_PD_VI>;
                clock-names = "xvclk";
                pinctrl-names = "default";
                flash-leds = <&flash_led>;
                pwdn-gpios = <&pca9555 PCA_IO0_1 GPIO_ACTIVE_HIGH>;
                firefly,clkout-enabled-index = <0>;
                rockchip,camera-module-index = <1>;
                rockchip,camera-module-facing = "front";
                rockchip,camera-module-name = "YT-RV1109-2-V1";
                rockchip,camera-module-lens-name = "40IR-2MP-F20";
                port {
                        gc2093_out: endpoint {
                                        remote-endpoint = <&dphy2_in>;
                                        data-lanes = <1 2>;
                        };
                };
        };

};
```

### csi2_dphy1/csi2_dphy2相关配置
csi2_dphy0与csi2_dphy1/csi2_dphy2互斥,不可同时使用
```shell
&csi2_dphy0 {
         status = "disabled";
};

&csi2_dphy1 {
        status = "okay";
        /*
        * dphy1 only used for split mode,
        * can be used  concurrently  with dphy2
        * full mode and split mode are mutually exclusive
        */
        ports {
                #address-cells = <1>;
                #size-cells = <0>;

                port@0 {
                        reg = <0>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        dphy1_in: endpoint@1 {
                                        reg = <1>;
                                        remote-endpoint = <&gc2053_out>;
                                        data-lanes = <1 2>;
                        };
                };

                port@1 {
                        reg = <1>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        dphy1_out: endpoint@1 {
                                        reg = <1>;
                                        remote-endpoint = <&isp0_in>;
                        };
                };
        };
};

&csi2_dphy2 {
        status = "okay";
        /*
        * dphy2 only used for split mode,
        * can be used  concurrently  with dphy1
        * full mode and split mode are mutually exclusive
        */
        ports {
                #address-cells = <1>;
                #size-cells = <0>;

                port@0 {
                        reg = <0>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        dphy2_in: endpoint@1 {
                                        reg = <1>;
                                        remote-endpoint = <&gc2093_out>;
                                        data-lanes = <1 2>;
                        };
                };

                port@1 {
                        reg = <1>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        dphy2_out: endpoint@1 {
                                        reg = <1>;
                                        remote-endpoint = <&mipi_csi2_input>;
                        };
                };
        };
};

&csi2_dphy_hw {
      status = "okay";
};

&mipi_csi2 {
        status = "okay";

        ports {
                #address-cells = <1>;
                #size-cells = <0>;

                port@0 {
                        reg = <0>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        mipi_csi2_input: endpoint@1 {
                                        reg = <1>;
                                        remote-endpoint = <&dphy2_out>;
                                        data-lanes = <1 2>;
                        };
                };

                port@1 {
                        reg = <1>;
                        #address-cells = <1>;
                        #size-cells = <0>;

                        mipi_csi2_output: endpoint@0 {
                                        reg = <0>;
                                        remote-endpoint = <&cif_mipi_in>;
                                        data-lanes = <1 2>;
                        };
                };
        };
};


&rkcif_mipi_lvds {
        status = "okay";
        port {
                cif_mipi_in: endpoint {
                        remote-endpoint = <&mipi_csi2_output>;
                        data-lanes = <1 2>;
                };
        };
};

&rkcif_mipi_lvds_sditf {
        status = "okay";
        port {
                mipi_lvds_sditf: endpoint {
                        remote-endpoint = <&isp1_in>;
                        data-lanes = <1 2>;
                };
        };
};

```

### isp相关配置
其中rkisp_vir0节点的`remote-endpoint`指向`dphy1_out`
```shell
&rkisp {
        status = "okay";
};

&rkisp_mmu {
        status = "okay";
};

&rkisp_vir0 {
        status = "okay";
        port {
                #address-cells = <1>;
                #size-cells = <0>;

                isp0_in: endpoint@0 {
                        reg = <0>;
                        remote-endpoint = <&dphy1_out>;
                };
        };
};

&rkisp_vir1 {
        status = "okay";

        port {
                reg = <0>;
                #address-cells = <1>;
                #size-cells = <0>;

                isp1_in: endpoint@0 {
                        reg = <0>;
                        remote-endpoint = <&mipi_lvds_sditf>;
                };
        };
};


&rkcif_mmu {
    status = "okay";
};

&rkcif {
    status = "okay";
};
```

## 软件相关目录
```shell
Linux Kernel-4.19
|-- arch/arm/boot/dts #DTS配置文件
|-- drivers/phy/rockchip
|-- phy-rockchip-mipi-rx.c #mipi dphy驱动
|-- phy-rockchip-csi2-dphy-common.h
|-- phy-rockchip-csi2-dphy-hw.c
|-- phy-rockchip-csi2-dphy.c
|-- drivers/media
|-- platform/rockchip/cif #RKCIF驱动
|-- platform/rockchip/isp #RKISP驱动
|-- dev #包含 probe、异步注册、clock、pipeline、 iommu及media/v4l2 framework
|-- capture #包含 mp/sp/rawwr的配置及 vb2,帧中断处理
|-- dmarx #包含 rawrd的配置及 vb2,帧中断处理
|-- isp_params #3A相关参数设置
|-- isp_stats #3A相关统计
|-- isp_mipi_luma #mipi数据亮度统计
|-- regs #寄存器相关的读写操作
|-- rkisp #isp subdev和entity注册
|-- csi #csi subdev和mipi配置
|-- bridge #bridge subdev,isp和ispp交互桥梁
|-- platform/rockchip/ispp #rkispp驱动
|-- dev #包含 probe、异步注册、clock、pipeline、 iommu及media/v4l2 framework
|-- stream #包含 4路video输出的配置及 vb2,帧中断处理
|-- rkispp #ispp subdev和entity注册
|-- params #TNR/NR/SHP/FEC/ORB参数设置
|-- stats #ORB统计信息
|-- i2
```

## 单目CAM-8MS1M/双目CAM-2MS2MF摄像头的使用
firefly已经配置好相应的dts，单目摄像头CAM-8MS1M和双目摄像头CAM-2MS2MF使用互斥，只需包含相应的dtsi文件即可使用单目摄像头CAM-8MS1M或双目摄像头CAM-2MS2MF

Linux 已经配置好各种组合的 dts 和 mk 文件，编译前按需选择，不用修改文件

Android 需要如下修改：

### 使用单目摄像头CAM-8MS1M
dts的配置默认使用单目摄像头
```shell
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
index 7e2a8b2..14fa027 100755
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
@@ -7,6 +7,15 @@
+#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
+//#include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
```
### 使用双目摄像头CAM-2MS2MF
```shell
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
index 7e2a8b2..14fa027 100755
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj.dts
@@ -7,6 +7,15 @@
- #include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
+//#include "rk3568-firefly-aioj-cam-8ms1m.dtsi"
- //#include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
+ #include "rk3568-firefly-aioj-cam-2ms2m.dtsi"
```

## Camera底层调试
使用v4l2-ctl抓取camera数据帧
```shell
v4l2-ctl --verbose -d /dev/video0 --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap=4 --set-selection=target=crop,flags=0,top=0,left=0,width=1920,height=1080 --stream-to=/data/out.yuv
```

把out.yuv文件拷贝出来通过ubuntu去查看
```shell
ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 out.yuv
```

## Android系统使用camera应用
Android系统使用camera的apk打开摄像头需要配置camera3_profiles*.xml，具体可参考Android SDK `hardware/rockchip/camera/etc/camera`目录下的文件

## Linux系统预览摄像头
Buildroot直接使用qcamera打开摄像头，可进行拍摄与录制，详细参考 [ Buildroot使用手册 ](https://wiki.t-firefly.com/zh_CN/Firefly-Linux-Guide/manual_buildroot.html)

Ubuntu 单目摄像头预览可以使用如下脚本：
```bash
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/aarch64-linux-gnu/gstreamer-1.0
WIDTH=1920
HEIGHT=1080
SINK=xvimagesink

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK &

wait
```
也可以使用之前提到的抓帧方法，抓到数据后用mpv播放：
```bash
mpv test.yuv --demuxer=rawvideo --demuxer-rawvideo-w=1920 --demuxer-rawvideo-h=1080
```

对于双目摄像头预览，则使用如下脚本：
```bash
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/aarch64-linux-gnu/gstreamer-1.0
WIDTH=640
HEIGHT=480
SINK=xvimagesink

gst-launch-1.0 v4l2src device=/dev/video14 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK &
gst-launch-1.0 v4l2src device=/dev/video5 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK &

wait
```

## IQ文件
raw摄像头支持的iq文件路径`external/camera_engine_rkaiq/iqfiles/isp21`, 与以前不一样的地方是iq文件不再采用`.xml`的方式，而是采用`.json`的方式。虽有提供xml转json的工具, 但isp20的xml配置转换后也不适用isp21。

若使用raw摄像头sensor，请留意isp21目录所支持的iq文件

