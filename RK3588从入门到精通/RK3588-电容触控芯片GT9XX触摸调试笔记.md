# 1. 简介
- [专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)
- 本文是基于RK3588平台，   电容触控芯片GT9XX触摸调试总结。
- 触摸芯片：电容触控芯片GT9271
- 驱动代码："kernel\drivers\input\touchscreen\gt9xx\gt9xx.c"(驱动用的是系统自带的驱动代码)

# 2. 电容触控芯片GT9XX触摸调试
## 2.1 调试总览，调试步骤分析
- 步骤 ①  先将gt9xx驱动添加进SDK编译规则
- 步骤 ②  dts配置
- 步骤 ③  触摸编译烧写，调试

 ## 2.2 调试过程
- **步骤 ①  ：将gt9xx驱动添加进kernel编译规则**

	```
	一.在Makefile添加：`obj-$(CONFIG_TOUCHSCREEN_GT9XX)		+= gt9xx/`
	
	Makefile所在路径："kernel\drivers\input\touchscreen\Makefile"
	
	二.在Kconfig添加：`config TOUCHSCREEN_GT9XX的说明
	Kconfig所在路径："kernel\drivers\input\touchscreen\Kconfig"
	
	三. 在 rockchip_defconfig中添加：CONFIG_TOUCHSCREEN_GT9XX=y
	```


- **步骤 ②  dts配置**
	
	```
	&i2c6 {
	    status = "okay";
	    pinctrl-names = "default";
	    pinctrl-0 = <&i2c6m0_xfer>;
	    clock-frequency = <400000>;
	  
	    gt9xx: gt9xx@14 {      
	        status = "okay";
	        compatible = "goodix,gt9xx";
	         reg = <0x14>;
	        pinctrl-names = "default";
	        pinctrl-0 = <&gt9xx_gpio>;
	        touch-gpio = <&gpio0 RK_PD3 IRQ_TYPE_LEVEL_HIGH>;
	        reset-gpio = <&gpio0 RK_PC6 GPIO_ACTIVE_HIGH>;
	        max-x = <1200>;
	        max-y = <1920>;
	        tp-size = <89>; 
	        tp-supply = <&vcc_lcd_mipi1>;
	
	        configfile-num = <1>;   
	    };
	};
	```
- **步骤 ③  触摸编译烧写，调试**

## 2.3 调试问题总结
- **当触摸点与屏幕响应点相反时：在gt9xx.c驱动源文件里修改：**

	方法一：根据触摸反馈调整下面值：

	```
	if (val == 89) {
	        m89or101 = TRUE;
	        gtp_change_x2y = TRUE;         //X,Y轴互换
	        gtp_x_reverse = TRUE;          //X轴反向
	        gtp_y_reverse = FALSE;         //Y轴反向
	}
	```

	方法二：或者在事件上报函数里修改：
	
	```
	input_report_abs(ts->input_dev, ABS_MT_POSITION_X, ts->abs_x_max-x);
	input_report_abs(ts->input_dev, ABS_MT_POSITION_Y, ts->abs_y_max-y);
	```

- **触摸不太精准，位置偏下一点点。**
解决办法：更换GT9271_Config_20170526.cfg文件的配置。
系统自带的cfg文件有点偏差，找屏幕厂商更换1200 * 1920的cfg文件

 ## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)