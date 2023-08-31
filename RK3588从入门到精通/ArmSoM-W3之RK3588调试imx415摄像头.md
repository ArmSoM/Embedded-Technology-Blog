# 一、环境
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)
- soc：rk3588
- sensor：imx415
- board: ArmSoM-W3
- linux：5.10

# 二、imx415简介
* 品牌：SONY
* 型号：IMX415
* 接口：MIPI CSI
![在这里插入图片描述](https://img-blog.csdnimg.cn/2e78cc0b2c374b7499c119ff615fc357.jpeg)

# 三、驱动移植
瑞芯微支持的摄像头，有个support list，

此次从该list中选择了IMX415

## 3.1 驱动源文件及对应脚本

RK提供的默认sdk里面已经将支持的所有摄像头驱动都添加到了内核，所以不需要移植该驱动了。

需确认下移植驱动对应的一些信息

* 源程序
  
``
3588_linux/3588_linux5.10_v1.0.5/kernel/drivers/media/i2c/imx415.c
3588_linux/3588_linux5.10_v1.0.5/kernel/drivers/media/i2c/Makefile
3588_linux/3588_linux5.10_v1.0.5/kernel/drivers/media/i2c/Kconfig
3588_linux/3588_linux5.10_v1.0.5/kernel/arch/arm64/configs/rockchip_linxu_defconfig
``

* Makefile脚本

``
obj-$(CONFIG_VIDEO_IMX415)	+= imx415.o
``

* Kconfig脚本

```
config VIDEO_IMX415
	tristate "Sony IMX415 sensor support"
	depends on I2C && VIDEO_V4L2 && VIDEO_V4L2_SUBDEV_API
	depends on MEDIA_CAMERA_SUPPORT
	help
	  This is a Video4Linux2 sensor driver for the Sony
	  IMX415 camera.

	  To compile this driver as a module, choose M here: the
	  module will be called imx415.
```

* 驱动对应的宏开关

``
CONFIG_VIDEO_IMX415=y
``

## 3.2 dts设备树

1）摄像头链接示意图
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab8ba1b25e5c44619b7f791beb0d60ea.png)

IMX415摄像头与SOC连接的主要的引脚有：i2c、rst、pwdn、mclk、MIPI Clk、MIPI DATA

2）电路图
![在这里插入图片描述](https://img-blog.csdnimg.cn/31ddefedd6e74c17a4898815f37acf70.png)


由电路图可知，几个关键引脚关系：

reset信号：gpio4 A0
power0 down信号：gpio1 B0
I2C通道：3
clock：CLK_MIPI_CAMARAOUT_M3

3）设备树节点

```
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2022 Rockchip Electronics Co., Ltd.
 *
 */

/ {
	compatible = "radxa,rock-5b", "rockchip,rk3588";

	camera_pwdn_gpio: camera-pwdn-gpio {
		status = "okay";
		compatible = "regulator-fixed";
		regulator-name = "camera_pwdn_gpio";
		regulator-always-on;
		regulator-boot-on;
		enable-active-high;
		gpio = <&gpio1 RK_PB0 GPIO_ACTIVE_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&cam_pwdn_gpio>;
	};

	clk_cam_24m: external-camera-clock-24m {
		status = "okay";
		compatible = "fixed-clock";
		clock-frequency = <24000000>;
		clock-output-names = "clk_cam_24m";
		#clock-cells = <0>;
	};
};

&i2c3 {
	status = "okay";

	imx415: imx415@1a {
		status = "okay";
		compatible = "sony,imx415";
		reg = <0x1a>;
		clocks = <&cru CLK_MIPI_CAMARAOUT_M3>;
		clock-names = "xvclk";
		pinctrl-names = "default";
		pinctrl-0 = <&mipim0_camera3_clk>;
		power-domains = <&power RK3588_PD_VI>;
		pwdn-gpios = <&gpio1 RK_PB0 GPIO_ACTIVE_HIGH>;
		reset-gpios = <&gpio4 RK_PA0 GPIO_ACTIVE_LOW>;
		rockchip,camera-module-index = <0>;
		rockchip,camera-module-facing = "back";
		rockchip,camera-module-name = "CMK-OT2022-PX1";
		rockchip,camera-module-lens-name = "IR0147-50IRC-8M-F20";
		port {
			imx415_out0: endpoint {
				remote-endpoint = <&mipidphy0_in_ucam0>;
				data-lanes = <1 2 3 4>;
			};
		};
	};

	camera_imx219: camera-imx219@10 {
		status = "disabled";
		compatible = "sony,imx219";
		reg = <0x10>;

		clocks = <&clk_cam_24m>;
		clock-names = "xvclk";

		rockchip,camera-module-index = <0>;
		rockchip,camera-module-facing = "back";
		rockchip,camera-module-name = "rpi-camera-v2";
		rockchip,camera-module-lens-name = "default";

		port {
			imx219_out0: endpoint {
				remote-endpoint = <&mipidphy0_in_ucam1>;
				data-lanes = <1 2>;
			};
		};
	};
};

&csi2_dphy0_hw {
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

			mipidphy0_in_ucam0: endpoint@1 {
				reg = <1>;
				remote-endpoint = <&imx415_out0>;
				data-lanes = <1 2 3 4>;
			};

			mipidphy0_in_ucam1: endpoint@2 {
				reg = <2>;
				remote-endpoint = <&imx219_out0>;
				data-lanes = <1 2>;
			};
		};

		port@1 {
			reg = <1>;
			#address-cells = <1>;
			#size-cells = <0>;

			csidphy0_out: endpoint@0 {
				reg = <0>;
				remote-endpoint = <&mipi2_csi2_input>;
			};
		};
	};
};

&mipi2_csi2 {
	status = "okay";

	ports {
		#address-cells = <1>;
		#size-cells = <0>;

		port@0 {
			reg = <0>;
			#address-cells = <1>;
			#size-cells = <0>;

			mipi2_csi2_input: endpoint@1 {
				reg = <1>;
				remote-endpoint = <&csidphy0_out>;
			};
		};

		port@1 {
			reg = <1>;
			#address-cells = <1>;
			#size-cells = <0>;

			mipi2_csi2_output: endpoint@0 {
				reg = <0>;
				remote-endpoint = <&cif_mipi2_in0>;
			};
		};
	};
};

&rkcif {
	status = "okay";
};

&rkcif_mipi_lvds2 {
	status = "okay";

	port {
		cif_mipi2_in0: endpoint {
			remote-endpoint = <&mipi2_csi2_output>;
		};
	};
};

&rkcif_mipi_lvds2_sditf {
	status = "okay";

	port {
		mipi_lvds2_sditf: endpoint {
			remote-endpoint = <&isp0_vir0>;
		};
	};
};

&rkcif_mmu {
	status = "okay";
};

&rkisp0 {
	status = "okay";
};

&isp0_mmu {
	status = "okay";
};

&rkisp0_vir0 {
	status = "okay";

	port {
		#address-cells = <1>;
		#size-cells = <0>;

		isp0_vir0: endpoint@0 {
			reg = <0>;
			remote-endpoint = <&mipi_lvds2_sditf>;
		};
	};
};

&pinctrl {
	camera {
		cam_pwdn_gpio: cam-pwdn-gpio {
			rockchip,pins = <1 RK_PB0 RK_FUNC_GPIO &pcfg_pull_up>;
		};
	};
};
```

设备树的信息最终转换成i2c_client,并传递给IMX415驱动 imx415_probe(){ compatible = "imx415";与驱动的 of_match_table 保持一致

```
rockchip,camera-module-index = <0>;
rockchip,camera-module-facing = "back";
rockchip,camera-module-name = "CMK-OT2022-PX1";
rockchip,camera-module-lens-name = "IR0147-50IRC-8M-F20";
```

匹配的是external\camera_engine_rkaiq\iqfiles\isp3x下面的iq文件

# 四、调试技能

## 4.1 开机log

```
root@linaro-alip:/# dmesg | grep imx415
[    2.547754] imx415 3-001a: driver version: 00.01.08
[    2.547767] imx415 3-001a:  Get hdr mode failed! no hdr default
[    2.547819] imx415 3-001a: Failed to get power-gpios
[    2.547826] imx415 3-001a: could not get default pinstate
[    2.547831] imx415 3-001a: could not get sleep pinstate
[    2.547850] imx415 3-001a: supply dvdd not found, using dummy regulator
[    2.547918] imx415 3-001a: supply dovdd not found, using dummy regulator
[    2.547945] imx415 3-001a: supply avdd not found, using dummy regulator
[    2.613843] imx415 3-001a: Detected imx415 id 0000e0
[    2.613890] rockchip-csi2-dphy csi2-dphy0: dphy0 matches m00_b_imx415 3-001a:bus type 5
[   18.386174] imx415 3-001a: set fmt: cur_mode: 3864x2192, hdr: 0
[   18.389067] imx415 3-001a: set exposure(shr0) 2047 = cur_vts(2250) - val(203)
```

## 4.2 查看IMX415设备

驱动加载成功后，会有以下信息
* 查看摄像头设备节点：
```
root@linaro-alip:/rockchip-test# ls /dev/video* -l
crw-rw----+ 1 root video 81,  0  8月  7 15:26 /dev/video0
crw-rw----+ 1 root video 81,  1  8月  7 15:26 /dev/video1
crw-rw----+ 1 root video 81, 10  8月  7 15:26 /dev/video10
crw-rw----+ 1 root video 81, 11  8月  7 15:26 /dev/video11
crw-rw----+ 1 root video 81, 12  8月  7 15:26 /dev/video12
crw-rw----+ 1 root video 81, 13  8月  7 15:26 /dev/video13
crw-rw----+ 1 root video 81, 14  8月  7 15:26 /dev/video14
crw-rw----+ 1 root video 81, 15  8月  7 15:26 /dev/video15
crw-rw----+ 1 root video 81, 16  8月  7 15:26 /dev/video16
crw-rw----+ 1 root video 81, 17  8月  7 15:26 /dev/video17
crw-rw----+ 1 root video 81, 18  8月  7 15:26 /dev/video18
crw-rw----+ 1 root video 81, 19  8月  7 15:26 /dev/video19
crw-rw----+ 1 root video 81,  2  8月  7 15:26 /dev/video2
crw-rw----+ 1 root video 81, 20  8月  7 15:26 /dev/video20
crw-rw----+ 1 root video 81,  3  8月  7 15:26 /dev/video3
crw-rw----+ 1 root video 81,  4  8月  7 15:26 /dev/video4
crw-rw----+ 1 root video 81,  5  8月  7 15:26 /dev/video5
crw-rw----+ 1 root video 81,  6  8月  7 15:26 /dev/video6
crw-rw----+ 1 root video 81,  7  8月  7 15:26 /dev/video7
crw-rw----+ 1 root video 81,  8  8月  7 15:26 /dev/video8
crw-rw----+ 1 root video 81,  9  8月  7 15:26 /dev/video9
lrwxrwxrwx  1 root root       7  8月  7 15:26 /dev/video-camera0 -> video11
-rw-rw----  1 root video      4  8月  7 15:26 /dev/video-dec0
-rw-rw----  1 root video      4  8月  7 15:26 /dev/video-enc0
```

## 4.3 查看sys文件系统中文件信息
内核会为摄像头在目录/sys/class/video4linux下分配设备信息描述文件
```
root@linaro-alip:grep imx415 /sys/class/video4linux/v*/name
root@linaro-alip:/rockchip-test# grep imx415 /sys/class/video4linux/v*/name
/sys/class/video4linux/v4l-subdev2/name:m00_b_imx415 3-001a
```
```
root@linaro-alip:/rockchip-test# grep "" /sys/class/video4linux/v*/name | grep mainpath
/sys/class/video4linux/video11/name:rkisp_mainpath
```

## 4.4 查看拓扑 media-ctl -d /dev/media0 -p

```
root@linaro-alip:/rockchip-test# media-ctl -d /dev/media0 -p
Media controller API version 5.10.110

Media device information
------------------------
driver          rkcif
model           rkcif-mipi-lvds2
serial
bus info
hw revision     0x0
driver version  5.10.110

Device topology
- entity 1: stream_cif_mipi_id0 (1 pad, 11 links)
            type Node subtype V4L flags 0
            device node name /dev/video0
        pad0: Sink
                <- "rockchip-mipi-csi2":1 [ENABLED]
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 5: stream_cif_mipi_id1 (1 pad, 11 links)
            type Node subtype V4L flags 0
            device node name /dev/video1
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 [ENABLED]
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 9: stream_cif_mipi_id2 (1 pad, 11 links)
            type Node subtype V4L flags 0
            device node name /dev/video2
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 [ENABLED]
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 13: stream_cif_mipi_id3 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video3
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 [ENABLED]
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 17: rkcif_scale_ch0 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video4
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 [ENABLED]
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 21: rkcif_scale_ch1 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video5
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 [ENABLED]
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 25: rkcif_scale_ch2 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video6
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 [ENABLED]
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 29: rkcif_scale_ch3 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video7
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 [ENABLED]
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 33: rkcif_tools_id0 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video8
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 [ENABLED]
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 []

- entity 37: rkcif_tools_id1 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video9
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 [ENABLED]
                <- "rockchip-mipi-csi2":11 []

- entity 41: rkcif_tools_id2 (1 pad, 11 links)
             type Node subtype V4L flags 0
             device node name /dev/video10
        pad0: Sink
                <- "rockchip-mipi-csi2":1 []
                <- "rockchip-mipi-csi2":2 []
                <- "rockchip-mipi-csi2":3 []
                <- "rockchip-mipi-csi2":4 []
                <- "rockchip-mipi-csi2":5 []
                <- "rockchip-mipi-csi2":6 []
                <- "rockchip-mipi-csi2":7 []
                <- "rockchip-mipi-csi2":8 []
                <- "rockchip-mipi-csi2":9 []
                <- "rockchip-mipi-csi2":10 []
                <- "rockchip-mipi-csi2":11 [ENABLED]

- entity 45: rockchip-mipi-csi2 (12 pads, 122 links)
             type V4L2 subdev subtype Unknown flags 0
             device node name /dev/v4l-subdev0
        pad0: Sink
                [fmt:SGBRG10_1X10/3864x2192 field:none
                 crop.bounds:(12,16)/3840x2160
                 crop:(12,16)/3840x2160]
                <- "rockchip-csi2-dphy0":1 [ENABLED]
        pad1: Source
                -> "stream_cif_mipi_id0":0 [ENABLED]
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad2: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 [ENABLED]
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad3: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 [ENABLED]
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad4: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 [ENABLED]
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad5: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 [ENABLED]
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad6: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 [ENABLED]
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad7: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 [ENABLED]
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad8: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 [ENABLED]
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad9: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 [ENABLED]
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 []
        pad10: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 [ENABLED]
                -> "rkcif_tools_id2":0 []
        pad11: Source
                -> "stream_cif_mipi_id0":0 []
                -> "stream_cif_mipi_id1":0 []
                -> "stream_cif_mipi_id2":0 []
                -> "stream_cif_mipi_id3":0 []
                -> "rkcif_scale_ch0":0 []
                -> "rkcif_scale_ch1":0 []
                -> "rkcif_scale_ch2":0 []
                -> "rkcif_scale_ch3":0 []
                -> "rkcif_tools_id0":0 []
                -> "rkcif_tools_id1":0 []
                -> "rkcif_tools_id2":0 [ENABLED]

- entity 58: rockchip-csi2-dphy0 (2 pads, 2 links)
             type V4L2 subdev subtype Unknown flags 0
             device node name /dev/v4l-subdev1
        pad0: Sink
                [fmt:SGBRG10_1X10/3864x2192@10000/300000 field:none
                 crop.bounds:(12,16)/3840x2160]
                <- "m00_b_imx415 3-001a":0 [ENABLED]
        pad1: Source
                -> "rockchip-mipi-csi2":0 [ENABLED]

- entity 63: m00_b_imx415 3-001a (1 pad, 1 link)
             type V4L2 subdev subtype Sensor flags 0
             device node name /dev/v4l-subdev2
        pad0: Source
                [fmt:SGBRG10_1X10/3864x2192@10000/300000 field:none
                 crop.bounds:(12,16)/3840x2160]
                -> "rockchip-csi2-dphy0":0 [ENABLED]
```

从entity 63信息中可以看到：

该Entity完整的名称是：m00_b_imx415 3-001a
是一个V4L2 subdev(Sub-Device) Sensor
对应的节点是 /dev/v4l-subdev2，应用程序（如v4l2-ctl）可以打开它，并进行配置
仅有一个输出（Source）节点，记为pad0
它的输出格式是 [fmt:SGBRG10_1X10/3864x2192@10000/300000 field:none
                 crop.bounds:(12,16)/3840x2160]，其中SBGGR10是一种mbus-code的简称
Source pad0 链接到"rockchip-csi2-dphy0"的pad0，并且当前的状态是 ENABLED。

media-ctl -d /dev/media1 -p
```
- entity 6: rkisp_mainpath (1 pad, 1 link)
            type Node subtype V4L flags 0
            device node name /dev/video11
        pad0: Sink
                <- "rkisp-isp-subdev":2 [ENABLED]
```

# 五、测试
确认调试成功后，使用gst-launch-1.0命令可直接看到 imx415摄像头输入
```
gst-launch-1.0 v4l2src device=/dev/video11 ! video/x-raw,format=NV12,width=3840,height=2160, framerate=30/1 ! xvimagesink
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e647f441a76487d87cfc31e0f24cf7b.jpeg#pic_center)

## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)