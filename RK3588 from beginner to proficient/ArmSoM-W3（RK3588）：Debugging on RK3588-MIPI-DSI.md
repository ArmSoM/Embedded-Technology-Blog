# ArmSoM-W3（RK3588）：Debugging on RK3588-MIPI-DSI

## 1. Introduction

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- This article summarizes MIPI panel debugging on the RK3588 platform. 

## 2. Environment

- Hardware: 
ArmSoM-W3 RK3588 dev board, MIPI-DSI panel (ArmSoM official accessory)

- Software:  
OS: ArmSoM-W3 Debian11

## 3. MIPI Panel Debugging

### 3.1 Overview, Step-by-Step Analysis

- Step 1: Turn on backlight first

- Step 2: Configure dsi1_panel node based on panel datasheet 

- Step 3: Enable corresponding dsi node for boot logo

- Step 4: Compile, flash, debug panel

### 3.2 DTS Configuration

```
#include "rk3588-evb.dtsi"   // included dsi1_panel node
```

#### 3.2.1 Step 1: Backlight Configuration

```
dsi1_backlight: dsi1-backlight {
		status = "okay";
		compatible = "pwm-backlight";
		pwms = <&pwm2 0 25000 0>;
		brightness-levels = <
			  0  20  20  21  21  22  22  23
			 23  24  24  25  25  26  26  27
			 27  28  28  29  29  30  30  31
			 31  32  32  33  33  34  34  35
			 35  36  36  37  37  38  38  39
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
			248 249 250 251 252 253 254 255
		>;
		default-brightness-level = <200>;
		enable-gpios = <&gpio2 RK_PC2 GPIO_ACTIVE_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&dsi1_backlight_en>;
	};
```

#### 3.2.2 Step 2: Configure Panel Node per Datasheet

- Panel on Power up Initialization Sequence Tutorial: [RK3588 MIPI Panel Debugging: RK3588-MIPI-DSI LCD Power up Initialization Sequence]
	
- Panel on Timing  Sequence Parameter Tutorial: [RK3588 MIPI Panel Debugging:  RK3588-MIPI-DSI Panel Configuration]

- Panel on dts Configuration:

	```
	&dsi1_panel {
		power-supply = <&vcc_lcd_mipi1>;  //Use GPIO to emulate a regulator
		reset-gpios = <&gpio2 RK_PC1 GPIO_ACTIVE_LOW>;
		backlight = <&dsi1_backlight>;
		pinctrl-names = "default";
		pinctrl-0 = <&dsi1_lcd_rst_gpio>;
	
		panel-init-sequence = [
				13 00 02 B0 01
				13 00 02 C0 26
				13 00 02 C1 10
				13 00 02 C2 0E
				13 00 02 C3 00
				13 00 02 C4 00
				13 00 02 C5 23
				13 00 02 C6 11
				13 00 02 C7 22
				13 00 02 C8 20
				13 00 02 C9 1E
				13 00 02 CA 1C
				13 00 02 CB 0C
				13 00 02 CC 0A
				13 00 02 CD 08
				13 00 02 CE 06
				13 00 02 CF 18
				13 00 02 D0 02
				13 00 02 D1 00
				13 00 02 D2 00
				13 00 02 D3 00
				13 00 02 D4 26
				13 00 02 D5 0F
				13 00 02 D6 0D
				13 00 02 D7 00
				13 00 02 D8 00
				13 00 02 D9 23
				13 00 02 DA 11
				13 00 02 DB 21
				13 00 02 DC 1F
				13 00 02 DD 1D
				13 00 02 DE 1B
				13 00 02 DF 0B
				13 00 02 E0 09
				13 00 02 E1 07
				13 00 02 E2 05
				13 00 02 E3 17
				13 00 02 E4 01
				13 00 02 E5 00
				13 00 02 E6 00
				13 00 02 E7 00
				13 00 02 B0 03
				13 00 02 BE 04
				13 00 02 B9 40
				13 00 02 CC 88
				13 00 02 C8 0C
				13 00 02 C9 07
				13 00 02 CD 01
				13 00 02 CA 40
				13 00 02 CE 1A
				13 00 02 CF 60
				13 00 02 D2 08
				13 00 02 D3 08
				13 00 02 DB 01
				13 00 02 D9 06
				13 00 02 D4 00
				13 00 02 D5 01
				13 00 02 D6 04
				13 00 02 D7 03
				13 00 02 C2 00
				13 00 02 C3 0E
				13 00 02 C4 00
				13 00 02 C5 0E
				13 00 02 DD 00
				13 00 02 DE 0E
				13 00 02 E6 00
				13 00 02 E7 0E
				13 00 02 C2 00
				13 00 02 C3 0E
				13 00 02 C4 00
				13 00 02 C5 0E
				13 00 02 DD 00
				13 00 02 DE 0E
				13 00 02 E6 00
				13 00 02 E7 0E
				13 00 02 B0 06
				13 00 02 C0 A5
				13 00 02 D5 1C
				13 00 02 C0 00
				13 00 02 B0 00
				13 00 02 BD 30
	
				13 00 02 F9 5C
				13 00 02 C2 14
				13 00 02 C4 14
				13 00 02 BF 15
				13 00 02 C0 0C
	
	
				13 00 02 B0 00
				13 00 02 B1 79
				13 00 02 BA 8F
	
				05 C8 01 11
				05 32 01 29
			];
	
			panel-exit-sequence = [
				05 00 01 28
				05 00 01 10
			];
	
			disp_timings1: panel-timings {    
				native-mode = <&dsi1_timing0>;
				dsi1_timing0: timing0 {
					clock-frequency = <159400000>;   //Configure based on the timing sequence parameter table in the panel datasheet.
					hactive = <1200>;
					vactive = <1920>;
					hfront-porch = <80>;
					hsync-len = <1>;
					hback-porch = <60>;
					vfront-porch = <35>;
					vsync-len = <1>;
					vback-porch = <25>;
					hsync-active = <0>;
					vsync-active = <0>;
					de-active = <0>;
					pixelclk-active = <1>;
				};
			};
	};

#### 3.2.3 Step 3: Enable dsi Node for Boot Logo

	```
	//Enable the backlight PWM node
	&pwm2 {
	    status = "okay";
	    pinctrl-names = "active";
	    pinctrl-0 = <&pwm2m2_pins>;
	};
	
	// a MIPI panel is connected to dsi1, and enables dsi1. 
	 &dsi1 {
	    status = "okay";
	};
	
	&mipi_dcphy1 {
	    status = "okay";
	};
	
	//By default dsi is configured on vp2 and vp3, here dsi is set to use vp3.The vp2 or vp3 can be selected based on the panel resolution. vp2 supports up to 4K resolution while vp3 only supports up to 2048x1536 resolution.
	&dsi1_in_vp2 {
	    status = "disabled";
	};
	
	&dsi1_in_vp3 {
	    status = "okay";
	};
	
	//Configure dsi1 to show boot logo.
	&route_dsi1 {
	    status = "okay";
	    connect = <&vp3_out_dsi1>;
	};
	```

#### 3.3 Debugging  Commands

View panel info:

```
cat /sys/kernel/debug/dri/0/summary 
```

## 4. Issue Summary

There are some issues will be encountered during debugging:

Initially ,the panel was dark for a long time, but measurement of the power voltage and waveforms were normal. 

Later, a comparison was done using another set of board and panel. 

Finally found that the panel connector was not soldered well, and the MIPI panel cable was too long which affected data transmission. After replacing with a shorter cable, the panel lit up.

![在这里插入图片描述](https://img-blog.csdnimg.cn/f263955ed6864757b36ae5beeeb6ebcd.jpeg#pic_center =600x)

## ArmSoM Product Introduction: [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum: [http://forum.armsom.org/](http://forum.armsom.org/)

