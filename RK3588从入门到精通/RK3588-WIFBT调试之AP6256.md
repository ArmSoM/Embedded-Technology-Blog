# 一、简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)
- AP6256是正基科技推出的一款低成本，低功耗的双模模块。是一款SDIO接口单通道802.11ac双频支持BT5.0 蓝牙WiFi二合一模块。
- Model：AP6256：SDIO WIFI + UART BT
- Chip：BCM43456
- WiFi：	2.4G&5G
- BT：5.0
- WIFI Throughput：up:196 Mbits/sec down: 187 Mbits/sec

# 二、 环境介绍


- 硬件环境：
ArmSoM-W3 RK3588开发板、Model：AP6256：SDIO WIFI + UART BT

- 软件版本：
OS：ArmSoM-W3 Debian11
# 三、DTS配置

## 3.1 配置SDIO

```bash
/* SDIO接口Wi-Fi专用配置：SDIO接口节点 */

&sdio {
	max-frequency = <150000000>;  /* sdio接口的最大频率,可调整 */
	supports-sdio;
	bus-width = <4>;    /* 4线模式，可调整1线模式 */
	disable-wp;
	cap-sd-highspeed;
	cap-sdio-irq;
	keep-power-in-suspend;
	mmc-pwrseq = <&sdio_pwrseq>;
	non-removable;
	num-slots = <1>;
	pinctrl-names = "default";
	pinctrl-0 = <&sdiom0_pins>;
	sd-uhs-sdr104;     /* 支持SDIO3.0 */
	status = "okay";
};
```
## 3.2 WIFI的配置
-  WIFI_REG_ON: Wi-Fi的电源使能PIN脚配置 ( 控制WIFI模块电源的GPIO )
- WL_REG_ON由sdio_pwrseq节点进行管理控制，不需要在wireless-wlan节点下面重复添加WIFI,poweren_gpio配置;

```bash
/* SDIO接口Wi-Fi专用配置： WIFI_REG_ON: Wi-Fi的电源使能PIN脚 */

sdio_pwrseq: sdio-pwrseq {
		compatible = "mmc-pwrseq-simple";
		clocks = <&hym8563>;
		clock-names = "ext_clock";
		pinctrl-names = "default";
		pinctrl-0 = <&wifi_enable_h>;
		reset-gpios = <&gpio0 RK_PC4 GPIO_ACTIVE_LOW>;  /*跟电源使能状态恰好相反：高有效为LOW，低有效则为HIGH。切记：这个配置跟下面的WIFI,poweren_gpio是互斥的，不能同时配置！！！*/
		
		 /*特别注意：reset-gpios的GPIO_ACTIVE 配置跟poweren_gpio配置的电源使能状态恰好是相反的*/
	};
	
	
/* SDIO接口Wi-Fi专用配置：WIFI_REG_ON脚的pinctrl的配置 */	

&pinctrl {
		sdio-pwrseq {
		wifi_enable_h: wifi-enable-h {
			rockchip,pins = <0 RK_PC4 RK_FUNC_GPIO &pcfg_pull_none>;  /* 对应上面的WIFI_REG_ON，关掉上下拉，防止不能拉高或拉低 */
		};
	};
}
```

 
- WIFI节点配置

```bash
/* Wi-Fi节点 */

wireless_wlan: wireless-wlan {
		compatible = "wlan-platdata";
		wifi_chip_type = "ap6256";        //模块名称
		pinctrl-names = "default";
		pinctrl-0 = <&wifi_host_wake_irq>;
		WIFI,host_wake_irq = <&gpio0 RK_PB2 GPIO_ACTIVE_HIGH>;     //WIFI模块唤醒CPU的
		//WIFI,poweren_gpio = <&gpio0 RK_PC4 GPIO_ACTIVE_HIGH>;    //控制WIFI模块电源的GPIO,配置了sdio_pwrseq就不需要再配置poweren_gpio 
		status = "okay";
	};
```

- WIFI,host_wake_irq的配置说明：
```bash
WIFI,host_wake_irq = <&gpio0 RK_PB2 GPIO_ACTIVE_HIGH>; 

/* WIFI_WAKE_HOST: Wi-Fi中断通知主控的PIN脚。
* 特别注意：确认下这个Wi-Fi pin脚跟主控的pin的硬件连接关系，直连的话就是GPIO_ACTIVE_HIGH;
* 如果中间加了一个反向管就要改成低电平GPIO_ACTIVE_LOW触发
*/
```

- WIFI_WAKE_HOST脚的pinctrl的配置:
```bash
&pinctrl {
		wireless-wlan {
		wifi_host_wake_irq: wifi-host-wake-irq {
			rockchip,pins = <0 RK_PB2 RK_FUNC_GPIO &pcfg_pull_down>;
		};
	};
}

/* 注意一般Wi-Fi的wake host pin都是高电平触发，
* 所以默认这里要配置为下拉; 如果客户的硬件设计
* 是反向的则要改为上拉，总之要初始化为与触发电平
* 相反的状态
*/
```

## 3.3 蓝牙的配置
- 以下UART相关的都要配置为实际使用的UART口的所对应PIN，注意RTS/CTS pin一定要按照SDK设计
接(具体接法参考7.3章节的UART描述)，很多客户反馈的异常都是因为这两个PIN脚没有接导致初始化
异常，下面假设蓝牙使用UART4：

```bash
bt_uart6: wireless_bluetooth: wireless-bluetooth {
		compatible = "bluetooth-platdata";
		clocks = <&hym8563>;                                     //外部时钟
		clock-names = "ext_clock"; 
		uart_rts_gpios = <&gpio1 RK_PA2 GPIO_ACTIVE_LOW>;        //uart的rts脚
		pinctrl-names = "default", "rts_gpio";
		pinctrl-0 = <&uart6m1_rtsn>;
		pinctrl-1 = <&uart6_gpios>;
		BT,reset_gpio    = <&gpio3 RK_PA6 GPIO_ACTIVE_HIGH>;     //蓝牙的复位脚
		BT,wake_host_irq = <&gpio0 RK_PC5 GPIO_ACTIVE_HIGH>;     //蓝牙模块唤醒CPU的GPIO
		status = "okay";
	};
	
&pinctrl {
		wireless-bluetooth {
		uart6_gpios: uart6-gpios {
			rockchip,pins = <1 RK_PA2 RK_FUNC_GPIO &pcfg_pull_none>;
			};
		};
	}
```

- 蓝牙对应的uart6配置
```bash
/* 打开对应的UART配置 */

&uart6 {
	pinctrl-names = "default";
	
	/* 这里配置对应主控UART的TX/RX/CTS PIN ，不要配置RTS PIN*/
	pinctrl-0 = <&uart6m1_xfer &uart6m1_ctsn>;
	status = "okay";
};
```
# 四 、内核配置

## 4.1 WIFI配置：kernel配置defconfig

- MK文件中定义的Kernel defconfig：
- kernel的defconfig对应到这个文件： "kernel\arch\arm64\configs\rockchip_linux_defconfig"
```bash
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=rockchip_linux_defconfig
```

```bash
cd kernel

make ARCH=arm64 menuconfig

make savedefconfig
```
选择：Device Drivers  --->  Network device support   --->   Wireless LAN  --->   Rockchip Wireless LAN support
![在这里插入图片描述](https://img-blog.csdnimg.cn/dbe70be45d724dc9ae16819698a82095.png)
Wi-Fi驱动可编译到内核或者ko方式， <font color="red" size="3">切记下面两个配置必须二选一，否则Wi-Fi无法加载！</font>
- KO 配置：[* ] build wifi ko modules

	```bash
	CONFIG_WIFI_BUILD_MODULE=y
	
	# CONFIG_WIFI_LOAD_DRIVER_WHEN_KERNEL_BOOTUP is not set
	```

- buildin 配置：[* ] Wifi load driver when kernel bootup

	```bash
	CONFIG_WIFI_LOAD_DRIVER_WHEN_KERNEL_BOOTUP=y
	
	# CONFIG_WIFI_BUILD_MODULE is not set
	```

	
	```
- buildin 只能选择一个型号，realtek 模组和 ap6xxx 模组不能同时选择为y，且realtek的也只能选择其
中一个；
- ap6xxx 和 cypress 也是互斥的，只能选择一个且如果选择ap6xxx，cypress的配置自动消失，去掉ap
配置，cypress自动出现；
- <font color="red" size="3">ko方式则可以选择多个Wi-Fi</font>
## 4.2 蓝牙配置：kernel配置defconfig
- 正基和海华的模块使用内核的默认CONFIG_BT_HCIUART 驱动：
	```bash
	cd kernel
	
	make ARCH=arm64 menuconfig
	
	make savedefconfig
	```
选择： Networking support   --->   Bluetooth subsystem support   --->  Bluetooth device drivers
![在这里插入图片描述](https://img-blog.csdnimg.cn/fef9779a49bc434ab3e9f62a65234978.png)

<font color="red" size="3">注意：配置完成后要保存到对应的defconfig</font>




# 五、Wi-Fi/BT的文件及更新及编译说明
## 5.1 查看板上生成的ko文件和firmware / nvram文件
- 正基/海华模组以AP6256为例：对应的Wi-Fi/BT的firmware在SDK中的位置：

	```bash
	external/rkwifibt/firmware/broadcom/AP6256/
	├── bt
	│ └── BCM4345C5.hcd
	└── wifi
	├── fw_bcm43456c5_ag.bin
	├── fw_bcm43456c5_ag_mfg.bin
	└── nvram_ap6256.txt
	```
- 经过编译规则编译后，对应的文件被拷贝到工程的output目录：(kernel4.19内核由system变更为vendor目录)

	```bash
	buildroot/output/rockchip_rk3xxxx/target/
	/system(vendor)/lib/modules/bcmdhd.ko #驱动ko（如果是ko编译的话）
	/system(vendor)/etc/firmware/fw_bcm43456c5_ag.bin #驱动firmware文件存放位置
	/system(vendor)/etc/firmware/fw_bcm43456c5_ag_mfg.bin #驱动firmware文件存放位置
	/system(vendor)/etc/firmware/nvram_ap6256.txt #驱动nvram文件存放位置
	/system(vendor)/etc/firmware/BCM4345C5.hcd #蓝牙firmware文件（如果有蓝牙功能）
	```

- 最终烧录到机器中后，Wi-Fi运行时所需的文件及存放位置：

	```bash
	/system(vendor)/lib/modules/bcmdhd.ko #驱动ko（如果是ko编译的话）
	/system(vendor)/etc/firmware/fw_bcm43456c5_ag.bin #驱动firmware文件存放位置
	/system(vendor)/etc/firmware/fw_bcm43456c5_ag_mfg.bin #驱动firmware文件存放位置
	/system(vendor)/etc/firmware/nvram_ap6256.txt #驱动nvram文件存放位置
	/system(vendor)/etc/firmware/BCM4345C5.hcd #蓝牙firmware文件（如果有蓝牙功能）
## 5.2 编译配置说明
- mk文件配置路径：3588_linux5.10_v1.0.5/device/rockchip/rk3588/BoardConfig-rk3588-pi5.mk
- 兼容正基和Realtek
- RK_WIFIBT_TTY这个参数根据蓝牙对应的串口来配置，此处蓝牙对应的是uart6

	```bash
	# Define WiFi BT chip
	# Compatible with Realtek and AP6XXX WiFi : RK_WIFIBT_CHIP=ALL_AP
	# Compatible with Realtek and CYWXXX WiFi : RK_WIFIBT_CHIP=ALL_CY
	# Single WiFi configuration: AP6256 or CYW43455: RK_WIFIBT_CHIP=AP6256
	
	export RK_WIFIBT_CHIP=ALL_AP
	# Define BT ttySX
	export RK_WIFIBT_TTY=ttyS6
	```
## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)