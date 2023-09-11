# 一、简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 本文介绍RK3588平台的Camera:MIPI-CSI调试之通路解析

- MIPI联盟，即移动产业处理器接口（Mobile Industry Processor Interface 简称MIPI）联盟。MIPI（移动产业处理器接口）是			MIPI联盟发起的为移动应用处理器制定的开放标准和一个规范。

	目的是把手机内部的接口如摄像头、显示屏接口、射频/基带接口等标准化，从而减少手机设计的复杂程度和增加设计灵活性。
	
- CSI & DSI

	 • <font color="red" size="3">CSI </font>( Camera Serial Interface )：摄像头接口
	

 	 • <font color="red" size="3">DSI</font> ( Display Serial Interface )：显示接口
# 二、 名词解释：

- <font color="red" size="3">ISP</font> ( Image Signal Processor )： 即图像信号处理模块， 主要作用是对前端图像传感器输出的信号做后期处理，依赖于 ISP 才能在不同的光学条件下都能较好的还原现场细节。

- <font color="red" size="3">VICAP</font>（ Video capture ）：视频捕获单元
# 三、RK3588 的camera通路:

## 多sensor支持：

- 单路硬件isp最多支持4路复用，isp复用情况支持分辨率如下：
- 2路复用：最大分辨率3840x2160，dts对应配置2路rkisp_vir设备。
- 3路或4路复用：最大分辨率2560x1536，dts对应配置3或4路rkisp_vir设备。
- 硬件支持最多采集7路sensor：6mipi + 1dvp，多sensor软件通路如下：

**下图是RK3588 camera连接链路示意图，可以支持7路camera。**

![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-CameraDebugging-RK3588-camera-channel.png)








# 四、 链路解析：
![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-CameraDebugging-RK3588-camera-channel-single.png)


- 图中：mipi camera2---> <font color="red" size="3">csi2_dphy1</font> ---> mipi2_csi2 ---> rkcif_mipi_lvds2--->rkcif_mipi_lvds2_sditf --->rkisp0_vir2
	
- 对应节点：imx415 ---> <font color="red" size="3">csi2_dphy0</font> ---> mipi2_csi2 ---> rkcif_mipi_lvds2--->rkcif_mipi_lvds2_sditf --->rkisp0_vir2
	
- 链接关系：sensor---> csi2 dphy---->mipi csi host--->vicap
	
- 实线链路解析： Camera sensor ---> dphy ---> 通过mipi_csi2模块解析mipi协议---> vicap <font color="red" size="3">( rkcif节点代表vicap )</font>
	
- 虚线链路解析：vicap ---> rkcif_mipi_lvds2_sditf --->  isp

	<font color="red" size="3">每个vicap节点与isp的链接关系，通过对应虚拟出的XXX_sditf来指明链接关系。</font>
	
	
	

# 五、RK3588硬件通路框图
![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/RK3588-CameraDebugging-Hardware-path-block-diagram.png)
- rk3588支持2个isp硬件，每个isp设备可虚拟出多个虚拟节点，软件上通过回读的方式，依次从ddr读取每一路的图像数据进isp处理。对于多摄方案，建议将数据流平均分配到两个isp上。

- 回读：指数据经过vicap采集到ddr，应用获取到数据后，将buffer地址推送给isp，isp再从ddr获取图像数据。

# 六、详细解析：

1. imx415  :   Camera sensor

2. csi2_dphy0 ： rk3588支持2个dphy硬件，这里我们称之为dphy0_hw/dphy1_hw ，，两个dphy硬件都可以工作在full mode 和split mode两种模式下。

  **当使用dphy0_hw:** 
    
- full mode：节点名称使用csi2_dphy0，最多支持4 lane。
	当dphy0_hw使用full mode时，链路需要按照csi2_dphy1这条链路来配置，<font color="red" size="3">但是节点名称csi2_dphy1需要修改为csi2_dphy0，</font>软件上是通过phy的序号来区分phy使用的模式。
	
- split mode：拆分成2个phy使用，分别为csi2_dphy1（使用0/1 lane）、csi2_dphy2(使用2/3 lane)，每个phy最多支持2 lane。

 **当使用dphy1_hw:**
	  

 - full mode：节点名称使用csi2_dphy3，最多支持4 lane。
	当dphy1_hw使用full mode时，链路需要按照csi2_dphy4这条链路来配置，<font color="red" size="3">但是节点名称csi2_dphy4需要修改为csi2_dphy3，</font>软件上是通过phy的序号来区分phy使用的模式。
	
-  split mode：拆分成2个phy使用，分别为csi2_dphy4（使用0/1 lane）、csi2_dphy5(使用2/3 lane)，每个phy最多支持2 lane。


3. dcphy：
rk3588支持两个dcphy，节点名称分别为csi2_dcphy0/csi2_dcphy1。每个dcphy硬件支持RX/TX同时使用，对于camera输入使用的是RX。支持DPHY/CPHY协议复用；需要注意的是同一个dcphy的TX/RX只能同时使用DPHY或同时使用CPHY。其他dcphy参数请查阅rk3588数据手册。

4. 使用上述mipi phy节点，需要把对应的物理节点配置上。
（csi2_dcphy0_hw/csi2_dcphy1_hw/csi2_dphy0_hw/csi2_dphy1_hw）

5. 每个mipi phy都需要一个csi2模块来解析mipi协议，节点名称分别为mipi0_csi2~mipi5_csi2。

6. rk3588所有camera数据都需要通过vicap，再链接到isp。rk3588仅支持一个vicap硬件，这个vicap支持同时输入6路mipi phy，及一路dvp数据，所以我们将vicap分化成rkcif_mipi_lvds~rkcif_mipi_lvds5、rkcif_dvp等7个节点，各个节点的绑定关系需要严格按照框图的节点序号配置。

7. 每个vicap节点与isp的链接关系，通过对应虚拟出的XXX_sditf来指明链接关系。

8. rk3588支持2个isp硬件，每个isp设备可虚拟出多个虚拟节点，软件上通过回读的方式，依次从ddr读取每一路的图像数据进isp处理。对于多摄方案，建议将数据流平均分配到两个isp上。

9. 直通与回读模式：
	•直通：指数据经过vicap采集，直接发送给isp处理，不存储到ddr。需要注意的是hdr直通时，只有短帧是真正的直通，长帧需要存在ddr，isp再从ddr读取。

	•回读：指数据经过vicap采集到ddr，应用获取到数据后，将buffer地址推送给isp，isp再从ddr获取图像数据。

	•在dts配置时，一个isp硬件，如果只配置一个虚拟节点，默认使用直通模式，如果配置了多个虚拟节点默认使用回读模式。
	
# 七、单路Camera的dts配置说明：( 以imx415摄像头为例 )
- **案例场景：这里使用的是csi2_dphy0的单路camera配置：**
- **链路配置： imx415 —> csi2_dphy0 —> mipi2_csi2 —> rkcif_mipi_lvds2—>rkcif_mipi_lvds2_sditf —>rkisp0_vir2**
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
# 八、 调试技巧
## 8.1 i2c设备的通用调试命令：查看设备是否挂载到i2c总线下：


```bash
i2cdetect -y 3
```


## 8.2 摄像命令
- **Linux系统摄像命令：**
	
```bash
gst-launch-1.0 v4l2src device=/dev/video11 ! video/x-raw,format=NV12,width=3840,height=2160, framerate=30/1 ! xvimagesink
```

- **Android系统：**

	Android系统自带相机APP。点击APP，看摄像画面是否正常显示。


## 8.3 imx415 相关的log信息
```bash
dmesg | grep imx415
```

## 8.4 查看拓扑结构

```bash
 media-ctl -d /dev/media0 -p
```
## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)