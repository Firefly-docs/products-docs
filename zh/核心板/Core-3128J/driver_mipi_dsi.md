# MIPI DSI 使用

## Config配置

在arch/arm/configs/fireprime_defconfig添加配置：

```
CONFIG_LCD_MIPI=y
CONFIG_MIPI_DSI=y
CONFIG_RK32_MIPI_DSI=y
```

## 驱动配置

### 新建DTS配置文件

在arch/arm/boot/dts/目录中新建dst配置文件，如lcd-xxx-mipi.dtsi。

### 添加DTS文件和关闭tve使能

在arch/arm/boot/dts/rk3128-fireprime.dts中添加#include "lcd-xxx-mipi.dtsi"，如果原来include了其他屏的DTS配置，注释掉它们。  
使用MIPI屏显示，需要关闭tve的使能：

```
&tve {
    status = "disabled";
    test_mode = <0>;
};
```

### 添加背光节点信息

在lcd-xxx-mipi.dtsi中添加背光节点信息。

```
backlight {
    compatible = "pwm-backlight";
    pwms = <&pwm0 0 25000>;
    rockchip,pwm_id= <1>;
    /* | dark(255-221) | light scale(220-0) | , scale_div=255*/
    brightness-levels = </*255 254 253 252 251 250 249 248 247 246 245 244 243 242 241 240 239 238 237 236 235 234 233 232 231 230 229 228 227
266 225 224 223 222 221 */220 219 218 217 216 215 214 213 212 211 210 209 208 207 206 205 204 203 202 201 200 199 198 197 196 195 194 193 192 191 190 189
188 187 186 185 184 183 182 181 180 179 178 177 176 175 174 173 172 171 170 169 168 167 166 165 164 163 162 161 160 159 158 157 156 155 154 153 152 151
150 149 148 147 146 145 144 143 142 141 140 139 138 137 136 135 134 133 132 131 130 129 128 127 126 125 124 123 122 121 120 119 118 117 116 115 114 113
112 111 110 109 108 107 106 105 104 103 102 101 100 99 98 97 96 95 94 93 92 91 90 89 88 87 86 85 84 83 82 81 80 79 78 77 76 75 74 73 72 71 70 69 68 67 66
65 64 63 62 61 60 59 58 57 56 55 54 53 52 51 50 49 48 47 46 45 44 43 42 41 40 39 38 37 36 35 34 33 32 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15
14 13 12 11 10 9 8 7 6 5 4 3 2 1 0>;
    default-brightness-level = <180>;
    enable-gpios = <&gpio1 GPIO_A7 GPIO_ACTIVE_HIGH>;
};
```
- 属性：
    + pwms属性：配置PWM，Firefly-RK3128 使用 pwm0，范例中的25000是PWM频率，enable-gpios 是背光使能脚。   
    + brightness-levels属性：配置背光亮度数组，最大值为255，配置暗区和亮区，并把亮区数组做255的比例调节。比如范例中暗区是255-221，亮区是220-0。  
    + default-brightness-level属性：开机时默认背光亮度，范围为0-255。  
    + enable-gpios属性：配置背光使能引脚。   

具体请参考kernel中的说明文档：   

Documentation/devicetree/bindings/video/backlight/pwm-backlight.txt

### 配置MIPI相关信息

在lcd-xxx-mipi.dtsi中添加MIPI配置信息

```
disp_mipi_init: mipi_dsi_init{
    compatible = "rockchip,mipi_dsi_init";
    rockchip,screen_init	= <1>;
    rockchip,dsi_lane	= <4>;
    rockchip,dsi_hs_clk	= <500>;
    rockchip,mipi_dsi_num	= <1>;
};
```

- 属性：    
    + rockchip,screen_init属性：0表示不需要特殊指令初始化显示屏，1,表示需要初始化指令。
    + rockchip,dsi_lane属性：数据lane的数量。
    + rockchip,dsi_hs_clk属性：配置hsclk。
    + rockchip,mipi_dsi_num：配置只用DSI接口的数量，单通道MIPI屏为1，双通道MIPI屏为2（fireprime只支持单通道MIPI屏，故配置成1即可）。

具体请参考kernel中的说明文档：

Documentation/devicetree/bindings/video/rockchip_mipidsi_lcd.txt

### 配置LCD引脚

在lcd-xxx-mipi.dtsi中添加引脚配置信息

```
disp_mipi_power_ctr: mipi_power_ctr {
    compatible = "rockchip,mipi_power_ctr";
    mipi_lcd_rst:mipi_lcd_rst{
        compatible = "rockchip,lcd_rst";
        rockchip,gpios = <&gpio2 GPIO_B0 GPIO_ACTIVE_LOW>;
        rockchip,delay = <10>;
    };
    /*
    mipi_lcd_en:mipi_lcd_en {
        compatible = "rockchip,lcd_en";
        rockchip,gpios = <&gpio0 GPIO_C1 GPIO_ACTIVE_HIGH>;
        rockchip,delay = <10>;
    };*/
};
```
    
disp_mipi_power_ctr分别有电源使能引脚mipi_lcd_en、片选引脚mipi_lcd_cs，复位引脚mipi_lcd_rst，可以根据显示屏做修改和删减。

### 配置初始化命令

在lcd-xxx-mipi.dtsi中添加初始化命令信息

```
disp_mipi_init_cmds: screen-on-cmds {
    rockchip,cmd_debug = <0>;
    compatible = "rockchip,screen-on-cmds";
    rockchip,on-cmds1 {
        compatible = "rockchip,on-cmds";
        rockchip,cmd_type = <LPDT>;
        rockchip,dsi_id = <0>;
        rockchip,cmd = <0x05 0x29>;
        rockchip,cmd_delay = <0>;
    };
    rockchip,on-cmds2 {
        compatible = "rockchip,on-cmds";
        rockchip,cmd_type = <LPDT>;
        rockchip,dsi_id = <0>;
        rockchip,cmd = <0x05 0x11>;
        rockchip,cmd_delay = <200>;
    };
};
```
    
当rockchip,screen_init为1时需要配置显示屏的初始化命令，初始化命令在节点disp_mipi_init_cmds中配置。

* rockchip,cmd_debug属性：打开可输出指令调试信息。
* rockchip,on-cmdsXX子节点：配置每条指令的信息。
* rockchip,cmd_type：数据传输模式，LPDT或HSDT。
* rockchip,dsi_id：指令传输的DSI接口，0为向DSI0（双通道MIPI屏时为左半屏）发送指令，1为向DSI1（双通道MIPI屏时为右半屏）发送指令,2为同时向两个DSI发送数据。
* rockchip,cmd：指令序列。其中第一个字节为DSI数据类型，第二个字节为REG，后面的字节为指令内容。
* rockchip,cmd_delay：发送指令后的延时，单位为ms。

### 配置显示时序

在lcd-xxx-mipi.dtsi中添加显示时序信息

```
disp_timings: display-timings {
    native-mode = <&timing0>;
    compatible = "rockchip,display-timings";
    timing0: timing0 {
        screen-type = <SCREEN_MIPI>;
        lvds-format = <LVDS_8BIT_2>;
        out-face    = <OUT_P666>;
        color-mode = <COLOR_RGB>;
        clock-frequency = <67000000>;
        hactive = <768>;
        vactive = <1024>;
        hsync-len = <64>;
        hback-porch = <56>;
        hfront-porch = <60>;
        vsync-len = <14>;
        vback-porch = <30>;
        vfront-porch = <36>;
        hsync-active = <0>;
        vsync-active = <0>;
        de-active = <0>;
        pixelclk-active = <1>;
        swap-rb = <0>;
        swap-rg = <0>;
        swap-gb = <0>;
    };
};
```
               
- 时序的在节点disp_timings配置
    + screen-type属性：显示屏类型，Firefly-RK3128只支持单通道MIPI屏，配置成SCREEN_MIPI即可。
    + lvds-format属性：无关选项。
    + out-face属性：配置颜色，可为OUT_P888（24位）、OUT_P666（18位）或者OUT_P565（16位）。
    + clock-frequency属性：屏时钟，单位Hz。

其他的时序属性参考下图：

![](../../../rk3128_img/Core-3128J/MIPI_DSI.png)

### dsihost配置

单MIPI屏，需要使能dsihost0，如：

```
&dsihost0 {
   	status = "okay";
};
```
