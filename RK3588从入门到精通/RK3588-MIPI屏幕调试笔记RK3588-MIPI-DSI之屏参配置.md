# 1. 简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 本文是基于RK3588平台，MIPI屏调试之屏参配置解析

# 2. 环境介绍


- 硬件环境：
ArmSoM-W3 RK3588开发板、MIPI-DSI显示屏( ArmSoM官方配件 )

- 软件版本：
OS：ArmSoM-W3 Debian11

# 3. 屏参dts配置
- 屏幕的时序参数表如下：
## signal timing 
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b89fd1ce07a4084bacb4505aae56899.png)
- 屏参对应的dts配置：
	```bash
	disp_timings1: display-timings {    
				native-mode = <&dsi1_timing0>;
				dsi1_timing0: timing0 {
					clock-frequency = <159400000>;  
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
	```
- 解析：
- Hactive：水平有效像素

- Vactive：垂直有效像素

- 水平总周期 Htotal = Hactive + + hfront-porch +hsync-len + hback-porch 

- 垂直总周期 Vtotal = Vactive + + vfront-porch + vsync-len + vback-porch

- clock-frequency： 提供给lcd的时钟频率,一般屏的规格书都会给出, 也可以通过计算得到。

      clock-frequency = 水平总周期 * 垂直总周期  * 帧率：

	`clock-frequency = （hactive + hfront + hsync-len + hback）* (vactive + vfront + vsync-len + vback) * fps`

- 名词解析

	```bash
	hactive：          	横向分辨率。
	vactive:            纵向分辨率。
	hsync-len          	行同步回扫时间。
	hback-porn：   		行同步后肩时间。
	hfront-porn:        行同步前肩时间。
	vsync-len：         帧同步回扫时间。
	vback-porch：       帧同步后肩时间。
	vfront-proch：      帧同步前肩时间。
	de-active:          DE 信号极性。
	hysnc-active：      行同步信号极性。
	vsync-active：      帧同步信号极性
	
	hback-porch/hfront-porch/hsync-len：水平同步信号
	
	vback-porch/vfront-porch/vsync-len：垂直同步信号
	```
## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)