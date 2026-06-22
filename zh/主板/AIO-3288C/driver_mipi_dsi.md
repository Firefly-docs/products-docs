# MIPI DSI 使用

## Config配置

在 `arch/arm/configs/firefly_defconfig` 添加配置：

```c
CONFIG_LCD_MIPI=y
CONFIG_MIPI_DSI=y
CONFIG_RK32_MIPI_DSI=y
```

## LCD引脚配置

引脚配置放在 `arch/arm/boot/dts/firefly-rk3288.dts` 中的 lcdc0 子节点 power_ctr 中，分别有电源使能引脚 lcd_en、片选引脚 lcd_cs，复位引脚 lcd_rst，可以根据显示屏做修改和删减。 如：

```dts
power_ctr: power_ctr {
    rockchip,debug = <0>;
    lcd_en:lcd_en {
        rockchip,power_type = <GPIO>;
        gpios = <&gpio7 GPIO_A3 GPIO_ACTIVE_HIGH>;
        rockchip,delay = <10>;
    };

    lcd_cs:lcd_cs {
       rockchip,power_type = <GPIO>;
       gpios = <&gpio7 GPIO_A4 GPIO_ACTIVE_HIGH>;
       rockchip,delay = <10>;
    };

     /*lcd_rst:lcd_rst {
         rockchip,power_type = <GPIO>;
         gpios = <&gpio3 GPIO_D6 GPIO_ACTIVE_HIGH>;
         rockchip,delay = <5>;
     };*/
}
```

## 驱动配置

### 新建 DTS 配置文件

在 `arch/arm/boot/dts/` 目录中新建 dts 配置文件，如 lcd-xxx-mipi.dtsi。

### 添加 DTS 文件  

在 `arch/arm/boot/dts/firefly-rk3288.dts` 中添加 #include "lcd-xxx-mipi.dtsi"，如果原来 include 了其他屏的DTS配置，注释掉它们。

### 添加背光节点信息

在 lcd-xxx-mipi.dtsi 中添加背光节点信息。

```dts
backlight {
    compatible = "pwm-backlight";
    pwms = <&pwm1 0 10000>;
    rockchip,pwm_id= <1>;
    /* | dark(255-221) | light scale(220-0) | , scale_div=255*/

    brightness-levels = <
       /*255 254 253 252 251 250 249 248 247 246 245 244 243 242 241 240 239 238 237 236 235 234 233 232 231 230 229 228 227 226 225
224 223 222 221 */
220 219 218 217 216 215 214 213 212 211 210 209 208 207 206 205 204 203 202 201 200 199 198 197 196 195 194 193 192 191 190 189 188 187
186 185 184 183 182 181 180 179 178 177 176 175 174 173 172 171 170 169 168 167 166 165 164 163 162 161 160 159 158 157 156 155 154 153 152 151 150 149
148 147 146 145 144 143 142 141 140 139 138 137 136 135 134 133 132 131 130 129 128 127 126 125 124 123 122 121 120 119 118 117 116 115 114 113 112 111
110 109 108 107 106 105 104 103 102 101 100 99 98 97 96 95 94 93 92 91 90 89 88 87 86 85 84 83 82 81 80 79 78 77 76 75 74 73 72 71 70 69 68 67 66 65 64
63 62 61 60 59 58 57 56 55 54 53 52 51 50 49 48 47 46 45 44 43 42 41 40 39 38 37 36 35 34 33 32 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13
12 11 10 9 8 7 6 5 4 3 2 1 0>;
    default-brightness-level = <128>;
    enable-gpios = <&gpio8 GPIO_A6 GPIO_ACTIVE_HIGH>;
};
```

* 属性:
    + pwms 属性：配置 PWM，Firefly-RK3288 使用 pwm1，范例中的 10000 是 PWM 频率。
    + brightness-levels 属性：配置背光亮度数组，最大值为 255，配置暗区和亮区，并把亮区数组做 255 的比例调节。比如范例中暗区是 255-221，亮区是 220-0。
    + default-brightness-level 属性：开机时默认背光亮度，范围为 0-255。
    + enable-gpios 属性：配置背光使能引脚。

具体请参考 kernel 中的说明文档：

`Documentation/devicetree/bindings/video/backlight/pwm-backlight.txt`

### 配置MIPI相关信息

```dts
disp_mipi_init: mipi_dsi_init{
    compatible = "rockchip,mipi_dsi_init";
    rockchip,screen_init    = <1>;
    rockchip,dsi_lane       = <4>;
    rockchip,dsi_hs_clk     = <1000>;
    rockchip,mipi_dsi_num   = <1>;
};
```

* 属性：
    + rockchip,screen_init 属性：0 表示不需要特殊指令初始化显示屏，1 表示需要初始化指令。
    + rockchip,dsi_lane 属性：数据 lane 的数量。
    + rockchip,dsi_hs_clk 属性：配置 hsclk。
    + rockchip,mipi_dsi_num：配置只用 DSI 接口的数量，即单通道 MIPI 屏为 1，双通道 MIPI 屏为 2。

具体请参考 kernel 中的说明文档：

`Documentation/devicetree/bindings/video/rockchip_mipidsi_lcd.txt`

### 配置初始化命令

* 当rockchip,screen_init 为 1 时需要配置显示屏的初始化命令，初始化命令在节点 disp_mipi_init_cmds 中配置。
    + rockchip,cmd_debug 属性：打开可输出指令调试信息。
    + rockchip,on-cmdsXX 子节点：配置每条指令的信息。
    + rockchip,cmd_type：数据传输模式，LPDT 或 HSDT。
    + rockchip,dsi_id：指令传输的 DSI 接口，0 为向 DSI0（双通道 MIPI 屏时为左半屏）发送指令，1 为向 DSI1（双通道 MIPI 屏时为右半屏）发送指令，2 为同时向两个 DSI 发送数据。
    + rockchip,cmd：指令序列。其中第一个字节为 DSI 数据类型，第二个字节为 REG，后面的字节为指令内容。
    + rockchip,cmd_delay：发送指令后的延时，单位为 ms。

### 配置显示时序

* 时序的在节点 disp_timings 配置。
    + screen-type 属性：显示屏类型，单通道 MIPI 屏时为 SCREEN_MIPI，双通道 MIPI 屏时为 SCREEN_DUAL_MIPI。
    + lvds-format 属性：无关选项。
    + out-face 属性：配置颜色，可为 OUT_P888（24位）、OUT_P666（18位）或者 OUT_P565（16位）。
    + clock-frequency 属性：屏时钟，单位 Hz。

其他的时序属性参考下图：

![](../../../rk3288_img/AIO-3288C/mipi_dsi.png)

### dsihost 配置

如果是双 MIPI 屏，需要使能 dsihost0 和 dsihost1，如：

```dts
&dsihost0 {
   status = "okay";
};
&dsihost1 {
   status = "okay";
};
```

如果是单 MIPI 屏，只需使能 dsihost0，如：

```dts
&dsihost0 {
   status = "okay";
};
```