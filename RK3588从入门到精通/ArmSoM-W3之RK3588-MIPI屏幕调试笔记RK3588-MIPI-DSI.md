# 一. 简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 本文是基于RK3588平台，MIPI屏调试总结。

# 二. 环境介绍

- 硬件环境：
ArmSoM-W3 RK3588开发板、MIPI-DSI显示屏( ArmSoM官方配件 )

- 软件版本：
OS：ArmSoM-W3 Debian11

# 三. MIPI屏幕调试
## 3.1 调试总览，调试步骤分析
- 步骤 ①  先将背光点亮
- 步骤 ②  根据屏幕的规格书配置dsi1_panel节点
- 步骤 ③  打开对应的dsi节点，开机logo
- 步骤 ④  编译烧写，调试屏幕
## 3.2 DTS配置

```
#include "rk3588-evb.dtsi"   //引用了dsi1_panel 节点
```

### 3.2.1 步骤 ①  背光配置：

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

### 3.2.2 步骤 ②  根据datasheet配置屏幕节点：

- 屏幕上电初始化时序的配置教程见 [RK3588-MIPI屏幕调试笔记:RK3588-MIPI-DSI之LCD上电初始化时序](https://blog.csdn.net/nb124667390/article/details/130727394)
	
- 屏幕的时序参数配置教程见 [RK3588-MIPI屏幕调试笔记:RK3588-MIPI-DSI之屏参配置](https://blog.csdn.net/nb124667390/article/details/130727354)

- 屏幕dts配置如下：
	```
	&dsi1_panel {
		power-supply = <&vcc_lcd_mipi1>;  //使用gpio模拟regulator
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
	
			disp_timings1: display-timings {    
				native-mode = <&dsi1_timing0>;
				dsi1_timing0: timing0 {
					clock-frequency = <159400000>;   //根据屏幕的时序参数表配置
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
	```

### 3.2.3 步骤 ③  打开对应的dsi节点，开机logo

	```
	//打开背光的pwm节点
	&pwm2 {
	    status = "okay";
	    pinctrl-names = "active";
	    pinctrl-0 = <&pwm2m2_pins>;
	};
	
	//在dsi1上接了一个mipi屏，这个配置开启dsi1
	 &dsi1 {
	    status = "okay";
	};
	
	&mipi_dcphy1 {
	    status = "okay";
	};
	
	//默认dsi配置vp2和vp3上面，这里配置为dsi使用vp3，可以根据屏的分辨率来确认使用vp2还是vp3，vp2支持4K，vp3只支持 2048x1536
	&dsi1_in_vp2 {
	    status = "disabled";
	};
	
	&dsi1_in_vp3 {
	    status = "okay";
	};
	
	//配置dsi1显示开机logo
	&route_dsi1 {
	    status = "okay";
	    connect = <&vp3_out_dsi1>;
	};
	```
## 3.3 调试命令

查看显示信息命令：

```
cat /sys/kernel/debug/dri/0/summary
```

# 四. 问题总结
调试过程中遇到的问题：一开始很久没亮，测量了电源电压和波形都正常。后来换了一组板子和屏幕做对比。最后是因为屏幕的座子没焊接好，二是接MIPI屏幕的排线太长会影响数据传输。换了一根短的线之后屏幕亮起。

![在这里插入图片描述](https://img-blog.csdnimg.cn/f263955ed6864757b36ae5beeeb6ebcd.jpeg#pic_center =600x)

## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)