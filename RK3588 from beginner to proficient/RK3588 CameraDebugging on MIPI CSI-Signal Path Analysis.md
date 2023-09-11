# RK3588 Camera: Debugging on MIPI CSI-Signal Path Analysis

# 1、Introduction

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- This article introduces the signal path analysis for RK3588  camera: MIPI CSI debugging.

- MIPI Alliance, namely the Mobile Industry Processor Interface Alliance. MIPI  is an open standard and specification initiated by the MIPI Alliance. 

  Its purpose is to standardize interfaces inside mobile phones such as camera, display, RF/baseband, etc, thereby reducing the complexity of mobile phone design and increasing design flexibility.

- CSI & DSI

  • <font color="red" size="3">CSI </font>( Camera Serial Interface )：Camera Interface

 	 • <font color="red" size="3">DSI</font> ( Display Serial Interface )：Display port

# 2、Terminology Explanation

- ISP (Image Signal Processor): The image signal processing module. Its main function is to post-process the signals output by the front-end image sensor, and it relies on the ISP to better restore scene details under different optical conditions.
- VICAP (Video capture): The video capture unit

# 3、RK3588 Camera Signal Path:

## Multi-sensor support:

- The single hardware ISP supports up to 4-channel multiplexing at most. The ISP multiplexing supports resolutions as follows:

- 2-channel multiplexing: maximum resolution of 3840x2160. The DTS corresponds to configuring 2 rkisp_vir devices.

- 3 or 4 channel multiplexing: maximum resolution of 2560x1536. The DTS corresponds to configuring 3 or 4 rkisp_vir devices.

- The hardware supports up to 7 sensor acquisitions at most: 6 MIPI + 1 DVP. The multi-sensor software path is as follows:

  

  The figure below is a schematic diagram of the RK3588 camera connection path, which can support 7 cameras.

  

![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-CameraDebugging-RK3588-camera-channel.png)








# 4、 Linkage Analysis：

![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-CameraDebugging-RK3588-camera-channel-single.png)


- In the picture：mipi camera2---> <font color="red" size="3">csi2_dphy1</font> ---> mipi2_csi2 ---> rkcif_mipi_lvds2--->rkcif_mipi_lvds2_sditf --->rkisp0_vir2

- Corresponding nodes：imx415 ---> <font color="red" size="3">csi2_dphy0</font> ---> mipi2_csi2 ---> rkcif_mipi_lvds2--->rkcif_mipi_lvds2_sditf --->rkisp0_vir2

- Linkage relationships：sensor---> csi2 dphy---->mipi csi host--->vicap

- Solid line linkage analysis： Camera sensor ---> dphy --->  analyzing MIPI protocol using mipi_csi2 module---> vicap <font color="red" size="3">(rkcif node represents vicap )</font>

- Dashed line linkage analysis：vicap ---> rkcif_mipi_lvds2_sditf --->  isp

  <font color="red" size="3">

  The linkage between each vicap node and isp is indicated by the corresponding virtually generated XXX_sditf.</font>

  

# 5、RK3588 Hardware Signal Path Block Diagram

![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-CameraDebugging-Hardware-path-block-diagram.png)

- RK3588 supports 2 ISP hardware units. Each ISP device can virtualize multiple virtual nodes. The software sequentially reads the image data of each path from DDR into the ISP for processing in a read-back manner.

  For multi-camera solutions, it is recommended to evenly distribute the data streams to the two ISPs.

- Readback: Data is captured to DDR via VICAP, then the application gets the data and pushes the buffer address to the ISP for it to retrieve image data from DDR.

# 6、Detailed Analysis：

1. imx415  :   Camera sensor

2. csi2_dphy0 ：RK3588 supports 2 dphy hardware units, which are referred to here as dphy0_hw/dphy1_hw. 

   The two dphy hardware units can work in both full mode and split mode.



  **When using dphy0_hw:** 

- Full mode: The node name uses csi2_dphy0, supporting up to 4 lanes at most. 

  When dphy0_hw uses full mode, the link needs to be configured according to the csi2_dphy1 link, **but the node name csi2_dphy1 needs to be changed to csi2_dphy0.** The software distinguishes the mode used by the phy based on the phy sequence number.

- Split mode: Split into 2 phys for use, namely csi2_dphy1 (using 0/1 lane) and csi2_dphy2 (using 2/3 lanes). Each phy supports up to 2 lanes at most.



**When using dphy1_hw:**

 - Full mode: The node name uses csi2_dphy3, supporting up to 4 lanes at most. 

   When dphy1_hw uses full mode, the link needs to be configured according to the csi2_dphy4 link, **but the node name csi2_dphy4 needs to be changed to csi2_dphy3.** The software distinguishes the mode used by the phy based on the phy sequence number. 

 -  Split mode: Split into 2 phys for use, namely csi2_dphy4 (using 0/1 lane) and csi2_dphy5 (using 2/3 lanes). Each phy supports up to 2 lanes at most.


3. dcphy：
   RK3588 supports two dcphys, with node names csi2_dcphy0/csi2_dcphy1 respectively. Each dcphy hardware supports concurrent RX/TX use. The RX is used for camera input. 

   It supports multiplexing of DPHY/CPHY protocols. Note that the TX/RX of the same dcphy can only use DPHY or CPHY at the same time. Please refer to the RK3588 datasheet for other dcphy parameters.

4. To use the above MIPI phy nodes, the corresponding physical nodes need to be configured.
   （csi2_dcphy0_hw/csi2_dcphy1_hw/csi2_dphy0_hw/csi2_dphy1_hw）

5. Each MIPI phy requires a csi2 module to parse the MIPI protocol. The node names are mipi0_csi2 to mipi5_csi2 respectively.

6. All RK3588 camera data needs to go through VICAP before linking to ISP. RK3588 only supports one VICAP hardware. This VICAP supports up to 6 MIPI phys and one DVP data input simultaneously. Therefore, we divide VICAP into 7 nodes: rkcif_mipi_lvds~rkcif_mipi_lvds5, rkcif_dvp. The binding relationship of each node needs to be configured strictly according to the node sequence number in the block diagram.

7. The linkage between each VICAP node and ISP is indicated by the corresponding virtually generated XXX_sditf.

8. RK3588 supports 2 ISP hardware units. Each ISP device can virtualize multiple virtual nodes. The software sequentially reads the image data of each path from DDR into the ISP for processing in a read-back manner. For multi-camera solutions, it is recommended to evenly distribute the data streams to the two ISPs.

9. Direct and readback modes:

   Direct: Data goes through VICAP and is sent directly to the ISP for processing without being stored in DDR. Note for HDR direct mode, only short frames are fully direct, long frames need to be stored in DDR for the ISP to read out.

   Readback: Data is captured to DDR via VICAP, then the application gets the data and pushes the buffer address to the ISP for it to retrieve image data from DDR.

   In DTS configuration, for one ISP hardware, direct mode is default if only one virtual node is configured. Readback mode is default if multiple virtual nodes are configured.

# 7、The DTS Configuration Instructions for Single Camera (using IMX415 camera as an example):

-  Use case scenario: Here the configuration uses a single camera link on csi2_dphy0:*
-  Link configuration: imx415 —> csi2_dphy0 —> mipi2_csi2 —> rkcif_mipi_lvds2—>rkcif_mipi_lvds2_sditf —>rkisp0_vir2

```bash
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

# 8、 Debugging Tips

## 8.1 Common debug commands for I2C devices： Check if the device is mounted under the I2C bus:


```bash
i2cdetect -y 3
```


## 8.2 Camera debugging commands

- Linux system camera commands：**

```bash
gst-launch-1.0 v4l2src device=/dev/video11 ! video/x-raw,format=NV12,width=3840,height=2160, framerate=30/1 ! xvimagesink
```

- **Android system：**

  Android system has a built-in camera app. Click on the app and check if the camera preview is displayed normally.


## 8.3 Imx415 Related Log Information

```bash
dmesg | grep imx415
```

## 8.4 Check Topology

```bash
 media-ctl -d /dev/media0 -p
```

## ArmSoM Product Introduction： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum： [http://forum.armsom.org/](http://forum.armsom.org/)