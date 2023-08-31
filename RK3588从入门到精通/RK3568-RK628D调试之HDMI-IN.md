# 一. 简介
- 本文是基于RK3568平台，HDMI-IN调试总结。  
- 视频桥接芯片：RK628D
- 驱动代码："kernel\drivers\media\i2c\rk628"(驱动用的是rk628-for-all-v21版本)
- 本次调试的方案功能：HDMI-IN信号通过RK628D转换成MIPI-CSI传到主控SOC
- 参考文档："RKDocs\common\RK628\Rockchip_RK628D_For_All_Porting_Guide_CN_V21.pdf"
- 场景描述：
    ① RK3568 不直接支持HDMI-IN接口，SOC有MIPI-CSI功能。需将HDMI-IN转换成MIPI-CSI才能获取视频信息。

	②此文章使用场景是一个HDMI-IN信号输入到RK628D芯片，RK628D再将HDMI-IN信号转换成MIPI-CSI信号输出到RK3588 SOC。通过软件抓取输入进SOC的视频信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d81c20dfe1a448b0ad0f2bbb7ce38673.png#pic_center)



# 二. 视频桥接芯片RK628D调试
## 2.1 RK628驱动介绍
-  RK628驱动有两个版本，一个是SDK系统自带的版本，一个是RK628-for-all版本。

-  RK628 分为 Display 通路和 HDMI IN 通路，SDK 版本 Display 通路基于DRM框架，HDMI IN 通路基于V4L2框架。

-  RK628-For-All 版本驱动一样也分为Display 通路和 HDMI IN 通路，Display 通路的驱动于drivers/misc/rk628/
下，HDMI IN 通路的驱动于drivers/media/i2c/rk628/下。本文采用RK628-For-All 版本HDMI IN 通路：media

- Media 为 RK628 HDMI IN 通路的驱动代码，将RK628D作为类camera设备使用，实现如下功能。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3dc698c0aae24c75acbb64fefc65405c.png)

## 2.2 调试总览，调试步骤分析
调试思路：先把四个HDMI-IN对应一个龙讯LT8641UXE芯片,对应一个RK628D芯片调好。再调另一组。

- 步骤 ①  移植驱动
- 步骤 ②  dts编写
- 步骤 ③  编译烧录，调试


## 2.3 移植驱动：
移植，向RK业务拿到移植的驱动文件：RK628-for-all-v21版本(和泰旨项目三是一样)
① config配置：Rockchip_defconfig:  关闭原SDK的628配置(不关闭的话会导致for_all驱动和sdk自带驱动冲突导致报错！！！)，开启for-all版本的628配置。

`# CONFIG_VIDEO_RK628_CSI is not set`     (不关闭的话会和media驱动有冲突，重复定义)
`CONFIG_VIDEO_RK628=y`                       (这是RK628-for-all media驱动开关配置)
CONFIG_DRM=y





## 2.4 dts编写
**链接关系：rk628 --> csi2_dphy0 --> rkisp_vir0 ( isp 链路 )**
```cpp
&i2c2 {
        clock-frequency = <400000>;
        status = "okay";
        pinctrl-names = "default";
 		pinctrl-0 = <&i2c2m1_xfer>;
        rk628_csi_v4l2: rk628_csi_v4l2@50 {
		status = "okay";
                reg = <0x50>;
                compatible = "rockchip,rk628-csi-v4l2";
                interrupt-parent = <&gpio0>;
                interrupts = <RK_PB0 IRQ_TYPE_LEVEL_HIGH>;
                enable-gpios = <&gpio3 RK_PA5 GPIO_ACTIVE_HIGH>;
                reset-gpios = <&gpio4 RK_PD2 GPIO_ACTIVE_LOW>;
                //hdcp-enable = <1>;
				scaler-en = <1>;

				pinctrl-names = "default";
        		pinctrl-0 = <&refclk_pins>;
				assigned-clocks = <&pmucru CLK_WIFI>;
                //assigned-clocks = <&pmucru 28>;
        		assigned-clock-rates = <24000000>;
				clocks = <&pmucru CLK_WIFI>;
				clock-names = "soc_24M";

                // pinctrl-names = "default";
                // pinctrl-0 = <&rk628_irq>;

                plugin-det-gpios = <&gpio0 15 GPIO_ACTIVE_LOW>;
                rockchip,camera-module-index = <0>;
                rockchip,camera-module-facing = "back";
                rockchip,camera-module-name = "RK628-CSI";
                rockchip,camera-module-lens-name = "NC";
               port {
                hdmiin_out0: endpoint {
                        remote-endpoint = <&mipi_in_ucam0>;
                        data-lanes = <1 2>;
                	};
        		};
        };
};

&csi2_dphy_hw {
        status = "okay";
};


&csi2_dphy0 {
	status = "okay";

	ports {
		#address-cells = <1>;
		#size-cells = <0>;
		port@0 {
			reg = <0>;
			#address-cells = <1>;
			#size-cells = <0>;

			mipi_in_ucam0: endpoint@0 {
					reg = <0>;
					remote-endpoint = <&hdmiin_out0>;
					data-lanes = <1 2>;
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


# 三. 调试命令：

- **查看media设备：**

	```cpp
	ls /dev/media*
	```
	```cpp
	/dev/media0  /dev/media1  
	```

- **i2c设备的通用调试命令：查看设备是否挂载到i2c总线下：**

	```cpp
	i2cdetect -y 2
	```

- **查看media节点的拓扑结构命令：**

	```cpp
	media-ctl -d /dev/media0 -p
	media-ctl -p
	```

- **抓图命令：**
	
	```cpp
	v4l2-ctl  -d   /dev/video0  --set-fmt-video=width=1920,height=1080,pixelformat=NV12 --stream-mmap=3 --stream-skip=100 --stream-to=/oem/NV12.yuv --stream-count=1 --stream-poll
	```
	
 ## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)