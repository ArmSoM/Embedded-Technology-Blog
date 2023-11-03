# 1. 简介
- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)

- 本⽂介绍在rockchip平台下如何配置i2c接口的方法并且添加调试验证i2c外设的例子

- 开发板：ArmSoM-W3

- Kernel：5.10.160

- OS：Debian11

  



# 2. i2c接口概述

i2c 总线控制器通过串行数据（SDA）线和串行时钟 （SCL）线在连接到总线的器件间传递信息。

i2c总线一些特征：

1. 只有两根线分别是串行数据线（SDA），串行时钟线（SCL）。

2. 每个器件都有一个唯一的地址识别

3. 使用串行8位双向数据传输方式。

4. 可以使用普通GPIO口模拟I2C，但要需要将GPIO配置成OD模式（开漏模式）

   

# 3. 芯片i2c资源

RK3588旗舰芯片上可使用的I2C有9组，ArmSoM SOM-3588-LGA核心板采用LGA 506引脚封装方式将I2C资源全部引出，ArmSoM-W3板子上接有**部分i2c外设以及40PIN资源**如下：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_RTC.png)

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_40PIN.png)





# 4. i2c使用

RK3588使用I2C 的驱动是i2c-rk3x.c，参考文件 kernel/Documentation/devicetree/bindings/i2c/i2c-rk3x.txt。



## 4.1 DTS配置

i2c资源使用只需要在设备树下进行配置，例如上述RTC芯片的配置如下：

```
&i2c6 {
	status = "okay";
	//i2c-scl-rising-time-ns = <265>;
	//i2c-scl-falling-time-ns = <11>;
	//clock-frequency = <400000>;

	hym8563: hym8563@51 {
		compatible = "haoyu,hym8563";
		reg = <0x51>;
		#clock-cells = <0>;
		clock-frequency = <32768>;
		clock-output-names = "hym8563";
		pinctrl-names = "default";
		pinctrl-0 = <&rtc_int>;
		interrupt-parent = <&gpio0>;
		interrupts = <RK_PB0 IRQ_TYPE_LEVEL_LOW>;
	};
};
```

参数说明：

- clock-frequency： 默认 frequency 为 100k 可不配置，其它 I2C 频率需要配置，最大可配置频率由i2c-scl-rising-time-ns 决定；例如配置 400k，clock-frequency=<400000>。
- i2c-scl-rising-time-ns：SCL 上升沿时间由硬件决定，例如测得 SCL 上升沿 365ns，i2c-scl-rising-time-ns=<365>。(默认可以不配置)
- i2c-scl-falling-time-ns: SCL 下降沿时间, 一般不变, 等同于 i2c-sda-falling-time-ns。(默认也可以不配置）

在使用i2c设备树配置的时候，有些方面需要注意：

1.上述rtc使用的引脚是I2C6_SDA_M0和I2C6_SCL_M0，硬件接口有些可以使用I2C6_SDA_M1，或者I2C6_SDA_M3，要修改默认配置

```
 i2c6: i2c@fec80000 {
  compatible = "rockchip,rk3588-i2c", "rockchip,rk3399-i2c";
  reg = <0x0 0xfec80000 0x0 0x1000>;
  clocks = <&cru 146>, <&cru 138>;
  clock-names = "i2c", "pclk";
  interrupts = <0 323 4>;
  pinctrl-names = "default";
  pinctrl-0 = <&i2c6m0_xfer>;//&i2c6m1_xfer、&i2c6m3_xfer
  #address-cells = <1>;
  #size-cells = <0>;
  status = "disabled";
 };
```

2. i2c地址主要由7bit的[二进制](https://so.csdn.net/so/search?q=二进制&spm=1001.2101.3001.7020)数值组成，最低位是读写标志位，**0表示写，1表示读**

   比如：读,0A3H   写，0A2H
   在linux驱动中要取这个ic设备的从设备地址，就是0xA3或者0xA2右移一位得到
   
   

## 4.2 GPIO 模拟 I2C

I2C 用 GPIO 模拟，内核已经有实现，请参考文档：Documentation/devicetree/bindings/i2c/i2c-gpio.txt
下面是使用的例子，dts 下配置 I2C 节点。

```
i2c@4 {
    compatible = "i2c-gpio";
    gpios = <&gpio5 9 GPIO_ACTIVE_HIGH>, /* sda */
    <&gpio5 8 GPIO_ACTIVE_HIGH>; /* scl */
    i2c-gpio,delay-us = <2>; /* ~100 kHz */
    #address-cells = <1>;
    #size-cells = <0>;
    pinctrl-names = "default";
    pinctrl-0 = <&i2c4_gpio>;
    status = "okay";
    
    gt9xx: gt9xx@14 {
        compatible = "goodix,gt9xx";
        reg = <0x14>;
        touch-gpio = <&gpio5 11 IRQ_TYPE_LEVEL_LOW>;
        reset-gpio = <&gpio5 10 GPIO_ACTIVE_HIGH>;
        max-x = <1200>;
        max-y = <1900>;
        tp-size = <911>;
        tp-supply = <&vcc_tp>;
        status = "okay";
      };
};
```

一般不推荐使用 GPIO，效率不高。



# 5. 检查i2c设备

## 5.1 IIC 第三方工具

I2C tool 是一个开源工具，需自行下载进行交叉编译，代码下载地址：
https://www.kernel.org/pub/software/utils/i2c-tools/或者<git clone git://git.kernel.org/pub/scm/utils/i2c-tools/i2c-tools.git>
编译后会生成 i2cdetect，i2cdump，i2cset，i2cget 等工具，可以直接在命令行上调试使用，I2C tool 是开源的，编译与使用参考里面的 README 与帮助说明。

ArmSoM-W3板子对应的出厂固件已经在系统下集成了这个工具，可以直接使用，比如扫描I2C总线上的RTC设备：

```
root@linaro-alip:~# i2cdetect -y 6
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- UU -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --                         
root@linaro-alip:~# 
```

扫描到对应的RTC芯片的I2C地址为0X51

常用的命令还有以下几个。

```
#检测当前系统有几组i2c总线
i2cdetect -l

#查看i2c-0接口上的设备
i2cdetect -a 6

#读取指定设备的全部寄存器的值。
i2cdump  -f -y 6 0x51

#读取指定IIC设备的某个寄存器的值，如下读取地址为0x51器件中的0x01寄存器值。
i2cget -f -y 6 0x51 0x01

#写入指定IIC设备的某个寄存器的值，如下设置地址为0x51器件中的0x01寄存器值为0x1a；
i2cset -f -y 3 0x51 0x01 0x1a
```



## 5.2 RTC使用

Linux系统下包含两个时间：系统时间和RTC时间。

linux命令中的date和time等命令都是用来设置系统时间的，而hwclock命令是用来设置和读写RTC时间的。

```
root@linaro-alip:~# hwclock -r
2018-05-24 16:38:13.115443+00:00 //查看硬件时间
root@linaro-alip:~# date
2018年 05月 24日 星期四 16:38:21 UTC //查看系统时间
root@linaro-alip:~# date -s "2023-10-24 11:45:00" 
2023年 10月 24日 星期二 11:45:00 UTC //重新设置系统时间
root@linaro-alip:~# hwclock -w //同步系统时间到rtc上，掉电不丢失时间
root@linaro-alip:~# hwclock -r
2023-10-24 11:45:17.694727+00:00
```



## 5.3 I2C 常见问题

如果调用 I2C 传输接口返回值为 -6(-ENXIO)时候，表示为 NACK 错误，即对方设备无应答响应

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_ERROR.png)

这种情况一般为外设的问题，常见的有以下几种情况：

- I2C 地址错误；

- I2C slave 设备处于不正常工作状态，比如没有上电，错误的上电时序以及设备异常等；

- I2C 时序不符合 slave 设备所要求也会产生 NACK 信号，比如 slave 设备需要的是 stop 信号,而不是

  repeat start 信号的时候；

- I2C 总线受外部干扰导致的，用示波器测量可以看到是一个 ACK 波形。

当出现 I2C 的 log 类似："timeout, ipd: 0x80, state: 1"时，看到 ipd 为 0x80 打印，可以说明当前 SCL 被
slave 拉住，要判断被哪个 slave 拉住：
	一是排除法，适用于外设不多的情况，而且复现概率高；
	二是需要修改硬件，在 SCL 总线上串入电阻，通过电阻两端产生的压差来确定，电压更低的那端
外设为拉低的 slave，电阻的选取以不影响 I2C 传输且可以看出压差为标准，一般上拉电阻的 1/20
以上都可以，如果是 host 拉低也可以看出。
常见的情况是 sda 被拉低，证明是谁拉低的。

有时候i2c初始化有问题时速率可以降低看有没有改善。遇到的 I2C 问题最好的办法是抓取 I2C 出错时候的波形，通过波形来分析 I2C 问
题，I2C 的波形非常有用，大部分的问题都能分析出来。



# 6. 读取eeprom数据实验

本章介绍通过IIC接口读写eeprom(AT24C08)的数据。 本次实验会以i2c-7做为示例，接其他i2c引脚操作也是一样的 当然，并不是只能用这个eeprom这个模组，这只是做个简单的示例，如果您没有这个模块，可以通过学习操作eeprom的方式操作您想要操作的i2c设备。

## 6.1 硬件连接

将eeprom接入到ArmSoM-W3开发板的i2c-7的总线上，如下图所示

<img src="C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_CONNECT.jpg" style="zoom:50%;" />

| 板子    | eeprom |
| ------- | ------ |
| 3.3V(1) | VCC    |
| GND(39) | GND    |
| SCL(5)  | SCL    |
| SDA(3)  | SCA    |



## 6.2 软件配置

在文件kernel\arch\arm64\boot\dts\rockchip\rk3588-armsom-w3.dts文件下添加下面代码：

```
&i2c7 {
	pinctrl-names = "default";
	pinctrl-0 = <&i2c7m3_xfer>;
	clock-frequency = <100000>;
	status = "okay";
	eeprom@50 {
                status = "okay";
				compatible = "at,24c08";
                reg = <0x50>;
        };
};
```

eeprom驱动在drivers/misc/eeprom/下面，如果是其他i2c接口芯片在kernel目录下没有驱动，可以去对找对应芯片厂商提供驱动文件

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_build.png)

将eeprom的驱动编译进内核测试



## 6.3 读写数据测试

找到模块位置：

```
root@linaro-alip:~# find / -name "at24"
/sys/bus/i2c/drivers/at24
```



读eeprom内容：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_read.png)



写eeprom内容：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ IIC篇 ]--IIC的开发指南\ArmSom-W3_I2C_write.png)



