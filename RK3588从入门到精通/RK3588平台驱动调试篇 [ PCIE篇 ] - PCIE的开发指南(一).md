RK3588平台驱动调试篇 [ PCIE篇 ] - PCIE的开发指南

# 1、PCIE接口概述

PCIe（Peripheral Component Interconnect Express）是一种用于连接计算机内部组件的高速接口标准。以下是关于PCIe接口的简要介绍：

- 高速传输： PCIe接口提供了高速的数据传输通道，可用于连接各种硬件设备，如图形卡、存储设备、网络适配器等。它的速度通常以每秒传输的数据位数（例如PCIe x1、x4、x8、x16等）来表示，每个通道的带宽可以根据需要扩展。

- 点对点连接： PCIe采用点对点连接的架构，这意味着每个设备都直接连接到主板上的PCIe插槽，而不需要与其他设备共享带宽。这有助于减少延迟并提高性能。

- 热插拔支持： PCIe接口支持热插拔，允许用户在计算机运行时添加或移除PCIe设备，而不需要重新启动计算机。

- 广泛应用： PCIe接口广泛用于连接图形卡、固态硬盘（SSD）、扩展卡、网络适配器和其他高性能设备。这使得计算机用户可以根据需要扩展和升级系统的性能和功能。


PCIe接口是一种计算机硬件连接标准，它提供了高速、高性能的数据传输通道，支持多种设备的连接。

# 2、传输速率简介

PCIe 分类、速度，按lane的个数分有 x1 x2 x4 x8 x16 （最大可支持32个通道），按代来分 有 gen1 gen2 gen3 gen4

![](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_1.png)

PCIe gen1 和 PCIe gen2 采用的编解码方式是 8b/10b，PCIe gen3 和 之后的 采用的是 128b/130b 的编码方式。

8b/10b 意思是说，当我们要传输8b的数据时，实际在通道上传输的是10b的数据，解码的时候，我们希望得到的是8b的有效数据。这样，相当于有效的带宽是实际带宽的 80%。

同理128b/130b，是传输128bit数据实际线路中传输的是130bit数据。

速率图中的单位间的关系：

传输速率单位 GT/s，表示 千兆传输/秒，是实际每秒传输的位数，他不包括额外吞吐量的开销位。

两个例子：

PCIe gen1 x1 传输速率 2.5GT/s = 2500MT/s = ( 2500 / 10 ) MB/s

PCIe gen3 x1 传输速率 8GT/s = 8000MT/s = ( 8000 / 130 ) x ( 128/8 ) MB/s= 984.6153... MB/s


**PCIe 可⽤带宽：吞吐量 = 传输速率 * 编码⽅案**

例如：PCIe 2.0 协议的每⼀条 Lane ⽀持58 / 10 = 4 Gbps = 500 MB/s 的速率，Pcie 2.0 x 8的通道为例，x8的可⽤带宽为 48 = 32 Gbps = 4 GB/s。



# 3、 芯片PCIE资源

## 3.1 硬件介绍

RK3588共有5个PCIe的控制器，硬件IP是⼀样的，配置不⼀样，其中⼀个4Lane DM模式可以⽀持作为EP使⽤，另外⼀个2Lane和3个1Lane控制器均只能作为RC使⽤。RK3588有两种PCIe PHY，其中⼀种为pcie3.0PHY，含2个Port共4个Lane，另⼀种是pcie2.0的PHY有3个，每个都是2.0 1Lane，跟SATA和USB combo使⽤。pcie3.0 PHY的4Lane可以根据实际需求拆分使⽤，拆分后需要合理配置对应的控制器。

![](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_2.png)

## 3.2 kernel dts解析之PCIe

控制器在DTS对应节点名称：

| 资源                   | 模式    | dts节点                      | 可用phy                     | 内部DMA |
| ---------------------- | ------- | ---------------------------- | --------------------------- | ------- |
| PCIe<br />Gen3 x 4lane | RC/EP   | pcie3x4:<br/>pcie@fe150000   | pcie30phy                   | 是      |
| PCIe<br/>Gen3 x 2lane  | RC only | pcie3x2:<br/>pcie@fe160000   | pcie30phy                   | 否      |
| PCIe<br/>Gen3 x 1lane  | RC only | pcie2x1l0:<br/>pcie@fe170000 | pcie30phy,<br/>combphy1_ps  | 否      |
| PCIe<br/>Gen3 x 1lane  | RC only | pcie2x1l1:<br/>pcie@fe180000 | pcie30phy,<br/>combphy2_psu | 否      |
| PCIe<br/>Gen3 x 1lane  | RC only | pcie2x1l2:<br/>pcie@fe190000 | combphy0_ps                 | 否      |

在kernel/arch/arm64/boot/dts/rockchip/rk3588.dtsi下有具体描述

**使用限制**

1. pcie30phy拆分后，pcie30x4控制器，⼯作于2Lane模式时只能固定配合pcie30phy的port0，⼯作于
   1Lane模式时，只能固定配合pcie30phy的port0lane0；
2. pcie30phy拆分后，pcie30x2控制器，⼯作于2Lane模式时只能固定配合pcie30phy的port1，⼯作于
   1Lane模式时，只能固定配合pcie30phy的port1lane0；
3. pcie30phy拆分为4个1Lane，pcie3phy的port0lane1只能固定配合pcie2x1l0控制器，pcie3phy的
   port1lane1只能固定配合pcie2x1l1控制器;
4. pcie30x4控制器⼯作于EP模式，可以使⽤4Lane模式，或者2Lane模式使⽤pcie30phy的port0，
   pcie30phy的port1中2lane可以作为RC配合其他控制器使⽤。默认使⽤common clock作为reference
   clock时，⽆法实现pcie30phy port0的lane0⼯作于EP模式，lane1⼯作于RC模式配合其他控制器使
   ⽤，因为Port0的两个lane是共⽤⼀个输⼊的reference clock，RC和EP同时使⽤clock可能会有冲突。
5. RK3588 pcie30phy 如果只使⽤其中⼀个port，另⼀个port也需要供电，refclk等其他信号可接地。



# 4、PCIe 使用配置

## 4.1 简介

Armsom-W3开发板上有 1 个 PCIe3.0 x 4 接口和一个PCIe2.0接口，如图

<img src="https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_11.jpg" style="zoom: 33%;" />

<img src="https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PICE_22.jpg" style="zoom: 33%;" />

可以插入对应模组使用, 如图：

<img src="https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_33.jpg" style="zoom:25%;" />

<img src="https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_44.jpg" style="zoom: 33%;" />

## 4.2 硬件设计

PCIe3.0 x 4 接口：

![](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_3.png)

PCIe2.0接口：

![](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_4.png)

## 4.3 软件配置

一般根据原理图在 DTS 中配置供电引脚、复位引脚，选择正确的 pcie 控制器节点和 PHY 节点使能就可以。

在**kernel/arch/arm64/boot/dts/rockchip/rk3588-armsom-w3.dts**中配置如下：

```
/ {
	vcc12v_dcin: vcc12v-dcin {
		compatible = "regulator-fixed";
		regulator-name = "vcc12v_dcin";
		regulator-always-on;
		regulator-boot-on;
		regulator-min-microvolt = <12000000>;
		regulator-max-microvolt = <12000000>;
	};

	vcc5v0_sys: vcc5v0-sys {
		compatible = "regulator-fixed";
		regulator-name = "vcc5v0_sys";
		regulator-always-on;
		regulator-boot-on;
		regulator-min-microvolt = <5000000>;
		regulator-max-microvolt = <5000000>;
		vin-supply = <&vcc12v_dcin>;
	};

	vcc3v3_pcie2x1l0: vcc3v3-pcie2x1l0 {
		compatible = "regulator-fixed";
		regulator-name = "vcc3v3_pcie2x1l0";
		regulator-min-microvolt = <3300000>;
		regulator-max-microvolt = <3300000>;
		enable-active-high;
		regulator-boot-on;
		regulator-always-on;
		gpios = <&gpio1 RK_PD2 GPIO_ACTIVE_HIGH>;
		startup-delay-us = <50000>;
		vin-supply = <&vcc5v0_sys>;
	};

	vcc3v3_pcie30: vcc3v3-pcie30 {
		compatible = "regulator-fixed";
		regulator-name = "vcc3v3_pcie30";
		regulator-min-microvolt = <3300000>;
		regulator-max-microvolt = <3300000>;
		enable-active-high;
		gpios = <&gpio1 RK_PA4 GPIO_ACTIVE_HIGH>;
		startup-delay-us = <5000>;
		vin-supply = <&vcc5v0_sys>;
	};

}

&pcie2x1l0 {
	reset-gpios = <&gpio4 RK_PA5 GPIO_ACTIVE_HIGH>;
	vpcie3v3-supply = <&vcc3v3_pcie2x1l0>;
	status = "okay";
};

&combphy1_ps {
	status = "okay";
};

&pcie30phy {
	rockchip,pcie30-phymode = <PHY_MODE_PCIE_AGGREGATION>;
	status = "okay";
};

&pcie3x4 {
	reset-gpios = <&gpio4 RK_PB6 GPIO_ACTIVE_HIGH>;
	vpcie3v3-supply = <&vcc3v3_pcie30>;
	status = "okay";
};
```

pcie30phy、combphy1_ps：PHY 节点

pcie3x4、pcie2x1l0：pcie3x4 控制器节点

reset-gpios：复位引脚属性

vcc3v3_pcie2x1l0、vcc3v3_pcie30：供电引脚节点



## 4.4 其他PCIE配置的实例

RK3588的控制器和PHY较多，按配置要点进⾏配置即可，这⾥还有⼏个典型范例供参考：
![](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/PCIE/PCIE_5.png)

## 4.4.1 ⽰例1 pcie3.0phy拆分2个2Lane RC, 3个PCIe 2.0 1Lane

```
/ {
    vcc3v3_pcie30: vcc3v3-pcie30 {
        compatible = "regulator-fixed";
        regulator-name = "vcc3v3_pcie30";
        regulator-min-microvolt = <3300000>;
        regulator-max-microvolt = <3300000>;
        enable-active-high;
        gpios = <&gpio3 RK_PC3 GPIO_ACTIVE_HIGH>;
        startup-delay-us = <5000>;
        vin-supply = <&vcc12v_dcin>;
    };
}；

&combphy0_ps {
	status = "okay";
};
&combphy1_ps {
	status = "okay";
};
&combphy2_psu {
	status = "okay";
};
&pcie2x1l0 {
    phys = <&combphy1_ps PHY_TYPE_PCIE>;
    reset-gpios = <&gpio4 RK_PA5 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie2x1l1 {
    phys = <&combphy2_psu PHY_TYPE_PCIE>;
    reset-gpios = <&gpio4 RK_PA2 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie2x1l2 {
    reset-gpios = <&gpio4 RK_PC1 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie30phy {
	/*pcie30phy的组合使⽤模式:
    PHY_MODE_PCIE_NANBNB  /* P1:PCIe3x2 + P0:PCIe3x2 */
    PHY_MODE_PCIE_NANBBI  /* P1:PCIe3x2 + P0:PCIe3x1*2 */
    PHY_MODE_PCIE_NABINB  /* P1:PCIe3x1*2 + P0:PCIe3x2 */
    PHY_MODE_PCIE_NABIBI  /* P1:PCIe3x1*2 + P0:PCIe3x1*2 */
	*/
    rockchip,pcie30-phymode = <PHY_MODE_PCIE_NANBNB>;
    status = "okay";
};
&pcie3x2 {
    reset-gpios = <&gpio4 RK_PB0 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie3x4 {
    num-lanes = <2>;//拆分为2lan使用
    reset-gpios = <&gpio4 RK_PB6 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
```



## 4.4.2 ⽰例2 pcie3.0phy拆分为4个1Lane, 1个使⽤PCIe 2.0 1 Lane

```
/ {
    vcc3v3_pcie30: vcc3v3-pcie30 {
        compatible = "regulator-fixed";
        regulator-name = "vcc3v3_pcie30";
        regulator-min-microvolt = <3300000>;
        regulator-max-microvolt = <3300000>;
        enable-active-high;
        gpios = <&gpio3 RK_PC3 GPIO_ACTIVE_HIGH>;
        startup-delay-us = <5000>;
        vin-supply = <&vcc12v_dcin>;
    };
}；
&combphy0_ps {
	status = "okay";
};
&pcie2x1l0 {
    phys = <&pcie30phy>;
    reset-gpios = <&gpio4 RK_PA5 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie2x1l1 {
    phys = <&pcie30phy>;
    reset-gpios = <&gpio4 RK_PA2 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie2x1l2 {
    reset-gpios = <&gpio4 RK_PC1 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie30phy {
    rockchip,pcie30-phymode = <PHY_MODE_PCIE_NABIBI>;
    status = "okay";
};
&pcie3x2 {
    num-lanes = <1>;
    reset-gpios = <&gpio4 RK_PB0 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
&pcie3x4 {
    num-lanes = <1>;
    reset-gpios = <&gpio4 RK_PB6 GPIO_ACTIVE_HIGH>;
    vpcie3v3-supply = <&vcc3v3_pcie30>;
    status = "okay";
};
```


pcie30phy拆分为4个1Lane时，port0lane0固定配合pcie3x4控制器，pcie3phy的port0lane1固定配合pcie2x1l0控制器，port1lane0固定配合pcie3x2控制器，pcie3phy的port1lane1固定配合pcie2x1l1控制器，加上combphy0_ps固定配合pcie2x1l2。

