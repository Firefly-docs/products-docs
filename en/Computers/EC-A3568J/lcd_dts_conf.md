```
/ {
	model = "AIO-3399PRO-JD4 Board lvds HSX101H40C (Android)";
	compatible = "rockchip,android", "rockchip,rk3399-firefly-lvds", "rockchip,rk3399";
};

&backlight {
        status = "okay";
        pwms = <&pwm0 0 25000 1>;
        enable-gpios = <&gpio1 1 GPIO_ACTIVE_HIGH>;
        default-brightness-level = <200>;
        polarity = <1>;
        brightness-levels = </*
                          0   1   2   3   4   5   6   7
                          8   9  10  11  12  13  14  15
                         16  17  18  19  20  21  22  23
                         24  25  26  27  28  29  30  31
                         32  33  34  35*/36  37  38  39
                         40  41  42  43  44  45  46  47
                         48  49  50  51  52  53  54  55
                         56  57  58  59  60  61  62  63
                         64  65  66  67  68  69  70  71
                         72  73  74  75  76  77  78  79
                         80  81  82  83  84  85  86  87
                         88  89  90  91  92  93  94  95
                         96  97  98  99 100 101 102 103
                        104 105 106 107 108 109 110 111
                        112 113 114 115 116 117 118 119
                        120 121 122 123 124 125 126 127
                        128 129 130 131 132 133 134 135
                        136 137 138 139 140 141 142 143
                        144 145 146 147 148 149 150 151
                        152 153 154 155 156 157 158 159
                        160 161 162 163 164 165 166 167
                        168 169 170 171 172 173 174 175
                        176 177 178 179 180 181 182 183
                        184 185 186 187 188 189 190 191
                        192 193 194 195 196 197 198 199
                        200 201 202 203 204 205 206 207
                        208 209 210 211 212 213 214 215
                        216 217 218 219 220 221 222 223
                        224 225 226 227 228 229 230 231
                        232 233 234 235 236 237 238 239
                        240 241 242 243 244 245 246 247
                        248 249 250 251 252 253 254 255>;
};

&dsi {
        status = "okay";
        dsi_panel: panel {
                compatible ="simple-panel-dsi";
                reg = <0>;
                //ddc-i2c-bu
                //power-supply = <&vcc_lcd>;
                //pinctrl-0 = <&lcd_panel_reset &lcd_panel_enable>;
                backlight = <&backlight>;
                /*
                enable-gpios = <&gpio1 1 GPIO_ACTIVE_LOW>;
                reset-gpios = <&gpio4 29 GPIO_ACTIVE_LOW>;
                */
                dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;
                dsi,format = <MIPI_DSI_FMT_RGB888>;
                //bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;
                dsi,lvds-force-clk = <800>; // 800/2/3 ~= 65Mhz
                dsi,lanes = <4>;

                dsi,channel = <0>;

                enable-delay-ms = <35>;
                prepare-delay-ms = <6>;

                unprepare-delay-ms = <0>;
                disable-delay-ms = <20>;

                size,width = <120>;
                size,height = <170>;

                status = "okay";

                panel-init-sequence = [
                            29 02 06 3C 01 09 00 07 00
                            29 02 06 14 01 06 00 00 00
                            29 02 06 64 01 0B 00 00 00
                            29 02 06 68 01 0B 00 00 00
                            29 02 06 6C 01 0B 00 00 00
                            29 02 06 70 01 0B 00 00 00
                            29 02 06 34 01 1F 00 00 00
                            29 02 06 10 02 1F 00 00 00
                            29 02 06 04 01 01 00 00 00
                            29 02 06 04 02 01 00 00 00
                            29 02 06 50 04 20 01 F0 03
                            29 02 06 54 04 19 00 5A 00                       //5A
                            29 02 06 58 04 20 03 24 00
                            29 02 06 5C 04 0A 00 19 00
                            29 02 06 60 04 00 05 0A 00
                            29 02 06 64 04 01 00 00 00
                            29 02 06 A0 04 06 80 44 00

                            29 02 06 A0 04 06 80 04 00
                            29 02 06 04 05 04 00 00 00
                            29 02 06 80 04 00 01 02 03
                            29 02 06 84 04 04 07 05 08
                            29 02 06 88 04 09 0A 0E 0F
                            29 02 06 8C 04 0B 0C 0D 10
                            29 02 06 90 04 16 17 11 12
                            29 02 06 94 04 13 14 15 1B
                            29 02 06 98 04 18 19 1A 06

                            29 02 06 9C 04 31 04 00 00
                ];
                panel-exit-sequence = [
                        05 05 01 28
                        05 78 01 10
                ];

                power_ctr: power_ctr {
                        rockchip,debug = <0>;
                        power_enable = <1>;
	                    bl_en:bl_en {
                                gpios = <&gpio0 1 GPIO_ACTIVE_HIGH>;
                                pinctrl-names = "default";
                                pinctrl-0 = <&lcd_panel_bl_en>;
                                rockchip,delay = <0>;
	                    };
	                    lcd_en:lcd_en {
                                gpios = <&gpio1 4 GPIO_ACTIVE_HIGH>;
                                pinctrl-names = "default";
                                pinctrl-0 = <&lcd_panel_lcd_en>;
                                rockchip,delay = <10>;
	                    };
                        lcd_pwr_en: lcd-pwr-en {
                                gpios = <&gpio0 12 GPIO_ACTIVE_HIGH>;
                                pinctrl-names = "default";
                                pinctrl-0 = <&lcd_panel_pwr_en>;
                                rockchip,delay = <10>;
                        };

                        lcd_rst: lcd-rst {
                                gpios = <&gpio1 13 GPIO_ACTIVE_HIGH>;
                                pinctrl-names = "default";
                                pinctrl-0 = <&lcd_panel_reset>;
                                rockchip,delay = <6>;
                        };
                };

                disp_timings: display-timings {
                        native-mode = <&timing0>;
                        timing0: timing0 {
                                clock-frequency = <166000000>; //166000000 @50
                                hactive = <800>;
                                vactive = <1280>;
                                hsync-len = <10>;   //20, 50
                                hback-porch = <100>; //50, 56
                                hfront-porch = <1580>;//50, 30 //1580
                                vsync-len = <10>;
                                vback-porch = <25>;
                                vfront-porch = <10>;
                                hsync-active = <0>;
                                vsync-active = <0>;
                                de-active = <0>;
                                pixelclk-active = <0>;
                        };
                };
        };
};
```

Power control pin of LCD is defined here:

```
bl_en:(GPIO1_C5)GPIO_ACTIVE_HIGH
lcd_en:(GPIO4_D6)GPIO_ACTIVE_HIGH
lcd_pwr_en:(GPIO1_A2)GPIO_ACTIVE_HIGH
lcd_rst:(GPIO4_D1)GPIO_ACTIVE_HIGH
```