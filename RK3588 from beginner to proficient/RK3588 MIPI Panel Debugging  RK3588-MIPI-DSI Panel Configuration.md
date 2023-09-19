# RK3588 MIPI Panel Debugging:  RK3588-MIPI-DSI Panel Configuration

# 1. Introduction

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- This article explains parameter configuration of MIPI panel debugging based on the RK3588.

# 2.  Environment


- Hardware: 
  ArmSoM-W3 RK3588 dev board, MIPI-DSI panel (ArmSoM official accessory)

- Software:  
  OS: ArmSoM-W3 Debian11

# 3. Panel Parameter on DTS Configuration 

- The timing sequence parameters  for panel are as follows:

## Signal Timing 

![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-MIPI-Panel-Debugging-RK3588-MIPI-DSI-Panel-Configuration.png)

- DTS Configuration：

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

  Analysis：

- Hactive:Horizontal active pixels

- Vactive:Vertical active pixels

- Horizontal total period  Htotal = Hactive + + hfront-porch +hsync-len + hback-porch 

- Vertical total period   Vtotal = Vactive + + vfront-porch + vsync-len + vback-porch

- Clock-Frequency：It provided to the LCD , usually specified in the panel datasheet, which can also be calculated.

      clock-frequency = Horizontal total * Vertical total  * Frame rate：

  `clock-frequency = （hactive + hfront + hsync-len + hback）* (vactive + vfront + vsync-len + vback) * fps`

- Term analysis:

  ```bash
  hactive: Horizontal resolution
  vactive: Vertical resolution
  hsync-len: Horizontal sync pulse width
  hback-porch: Back porch after HS pulse
  hfront-porch: Front porch before HS pulse 
  vsync-len: Vertical sync pulse width
  vback-porch: Back porch after VS pulse
  vfront-porch: Front porch before VS pulse
  de-active: DE signal polarity
  hsync-active: HS signal polarity
  vsync-active: VS signal polarity
  
  hback-porch/hfront-porch/hsync-len: Horizontal sync signals
  
  vback-porch/vfront-porch/vsync-len: Vertical sync signals 
  ```

## ArmSoM Product Introduction: [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum: http://forum.armsom.org/