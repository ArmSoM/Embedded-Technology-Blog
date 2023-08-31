##### RK3588平台驱动调试篇 [ GPIO篇 ] - RK3588-对GPIO的操作控制

# 1. 简介
- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)
- 本⽂介绍Linux操作gpio⽅法
- 开发板：ArmSoM-W3

# 2. GPIO配置
Rockchip Pin的ID按照 控制器(bank)+端口(port)+索引序号(pin) 组成

## 2.1 GPIO驱动介绍
驱动包括Pinctrl驱动（ drivers/pinctrl/pinctrl-rockchip.c ） 和 GPIO驱动
（ drivers/gpio/gpio-rockchip.c ）

 - [ ] Pinctrl驱动是主要驱动，提供IO的⽅法集
 - [ ] GPIO驱动是完成 gpiochip 的功能，包括 GPIO 和 IRQ

## 2.2 GPIO复用
RK3588有5个GPIO控制器，每个控制器可以控制32个IO，作为GPIO功能时，端口⾏为由GPIO控制器寄存器配置
- 控制器和GPIO控制器数量⼀致
- 端口固定 A、B、C和D
- 索引序号固定 0、1、2、3、4、5、6、7

同⼀个控制器如果存在多种复⽤引脚，⼀般叫做m0、m1、m2等等，比如PWM功能：

```
	pwm1 {
		/omit-if-no-ref/
		pwm1m0_pins: pwm1m0-pins {
			rockchip,pins =
				/* pwm1_m0 */
				<0 RK_PC0 3 &pcfg_pull_none>;
		};

		/omit-if-no-ref/
		pwm1m1_pins: pwm1m1-pins {
			rockchip,pins =
				/* pwm1_m1 */
				<1 RK_PD3 11 &pcfg_pull_none>;
		};

		/omit-if-no-ref/
		pwm1m2_pins: pwm1m2-pins {
			rockchip,pins =
				/* pwm1_m2 */
				<1 RK_PA3 11 &pcfg_pull_none>;
		};
	};
```
一个Pin脚可以复⽤成多种功能，比如pwm1的GPIO0-C0脚可以有以下脚的复用

> PDM0_CLK0_M1/PWM1_M0/I2C2_SDA_M0/CAN0_RX_M0/SPI0_MOSI_M0/PCIE30X1_0_CLKREQN_M0/GPIO0_C0_d


## 2.3 DTS介绍
 dts⼀般把pinctrl节点放在soc.dtsi，例如rk3588s.dtsi，⼀般位于最后⼀个节点，在这个文件中可以找到板子所有可以配置的功能引脚


```
aliases {
		csi2dcphy0 = &csi2_dcphy0;
		csi2dcphy1 = &csi2_dcphy1;
		csi2dphy0 = &csi2_dphy0;
		csi2dphy1 = &csi2_dphy1;
		csi2dphy2 = &csi2_dphy2;
		dsi0 = &dsi0;
		dsi1 = &dsi1;
		ethernet1 = &gmac1;
		gpio0 = &gpio0;
		gpio1 = &gpio1;
		gpio2 = &gpio2;
		gpio3 = &gpio3;
		gpio4 = &gpio4;
		i2c0 = &i2c0;
		i2c1 = &i2c1;
		i2c2 = &i2c2;
		i2c3 = &i2c3;
		i2c4 = &i2c4;
		i2c5 = &i2c5;
		i2c6 = &i2c6;
		i2c7 = &i2c7;
		i2c8 = &i2c8;
		rkcif_mipi_lvds0= &rkcif_mipi_lvds;
		rkcif_mipi_lvds1= &rkcif_mipi_lvds1;
		rkcif_mipi_lvds2= &rkcif_mipi_lvds2;
		rkcif_mipi_lvds3= &rkcif_mipi_lvds3;
		rkvenc0 = &rkvenc0;
		rkvenc1 = &rkvenc1;
		jpege0 = &jpege0;
		jpege1 = &jpege1;
		jpege2 = &jpege2;
		jpege3 = &jpege3;
		serial0 = &uart0;
		serial1 = &uart1;
		serial2 = &uart2;
		serial3 = &uart3;
		serial4 = &uart4;
		serial5 = &uart5;
		serial6 = &uart6;
		serial7 = &uart7;
		serial8 = &uart8;
		serial9 = &uart9;
		spi0 = &spi0;
		spi1 = &spi1;
		spi2 = &spi2;
		spi3 = &spi3;
		spi4 = &spi4;
		spi5 = &sfc;
		hdcp0 = &hdcp0;
		hdcp1 = &hdcp1;
	};
```
最后arch/arm64/boot/dts/rockchip/rk3588s-pinctrl.dtsi ⽂件通过include形式加到rk3588s.dtsi。
rk3588s-pinctrl.dtsi⽂件已经枚举了rk3588s芯⽚所有iomux的实例，各模块⼀般不再需要创建iomux实例；
创建iomux实例需要遵循如下规则：
1. 必须在pinctrl节点下
2. 必须以function+group的形式添加
3. function+group的格式如下

```
function {
	group {
		rockchip,pin = <bank gpio func &ref>;
		};
};
```
比如我添加如下gpio口的使用说明

```
&pinctrl {
	leds {
		led_rgb_b: led-rgb-b {
			rockchip,pins = <0 RK_PB7 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
	hym8563 {
		rtc_int: rtc-int {
			rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
};
```
**使用某一个gpio口的时候要注意是否有其他功能引用这个io口**

### 2.3.1 修改gpio口
如果我们需要修改pwm1的默认脚gpio0c0为gpio1d3

```
pwm1: pwm@fd8b0010 {
  compatible = "rockchip,rk3588-pwm", "rockchip,rk3328-pwm";
  reg = <0x0 0xfd8b0010 0x0 0x10>;
  #pwm-cells = <3>;
  pinctrl-names = "active";
  pinctrl-0 = <&pwm1m0_pins>;
  clocks = <&cru 677>, <&cru 676>;
  clock-names = "pwm", "pclk";
  status = "disabled";
 };
 &pwm1 {
 status = "okay";
};
```

```
&pwm1 {
 pinctrl-0 = <&pwm1m1_pins>;
 status = "okay";
};
```
### 2.3.2 配置某个GPIO电平
有个别需求是某个GPIO不属于某个特定模块，更多是某个电源开关，希望在系统开机后尽快输出⾼或低电平，使⽤"regulator-fixed"，regulator-fixed通常⽤于定义电压固定的regulator，或由某个GPIO开关控制的regulator。
以MIPI屏幕电源使能为例，gpio口为gpio1c4

```
vcc_lcd_mipi1: vcc-lcd-mipi1 {   
		status = "okay";
		compatible = "regulator-fixed";
		regulator-name = "vcc_lcd_mipi1";
		gpio = <&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>;
		enable-active-high;
		regulator-boot-on;
		regulator-state-mem {
			regulator-off-in-suspend;
		};
	};
	
```

 

# 3. Linux下控制GPIO

编译内核的时候加入 Device Drivers-> GPIO Support ->[*] /sys/class/gpio/… (sysfs interface)。

![image-20230829144412307](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230829144412307.png)

通过sysfs方式控制GPIO，先访问/sys/class/gpio目录，向export文件写入GPIO编号，使得该GPIO的操作接口从内核空间暴露到用户空间，GPIO的操作接口包括direction和value等，direction控制GPIO方向，而value可控制GPIO输出或获得GPIO输入

/sys/class/gpio 的使用说明可以参考这篇文章：[linux系统基于syfs控制gpio](https://blog.csdn.net/nb124667390/article/details/131104841)



文件IO方式操作GPIO，使用到了4个函数open、close、read、write。以下是一个简单的基于 C 语言的流水灯和呼吸灯效果的示例代码。这个示例代码使用的是 Linux 上的用户空间 GPIO 控制，你需要适配代码以使用正确的 GPIO 引脚和路径。

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

#define LED_NUM 4

// GPIO 控制的相关路径
#define SYSFS_GPIO_EXPORT "/sys/class/gpio/export"
#define SYSFS_GPIO_DIR_PREFIX "/sys/class/gpio/gpio"
#define SYSFS_GPIO_VALUE_SUFFIX "/value"

// 设置 GPIO 方向
void set_gpio_direction(int gpio, const char *dir) {
    char gpio_path[50];
    sprintf(gpio_path, "%s%d/direction", SYSFS_GPIO_DIR_PREFIX, gpio);
    
    int fd = open(gpio_path, O_WRONLY);
    if (fd == -1) {
        perror("Error opening direction file");
        exit(EXIT_FAILURE);
    }
    write(fd, dir, strlen(dir));
    close(fd);
}

// 控制 GPIO 输出
void set_gpio_value(int gpio, int value) {
    char gpio_path[50];
    sprintf(gpio_path, "%s%d/value", SYSFS_GPIO_DIR_PREFIX, gpio);
    
    int fd = open(gpio_path, O_WRONLY);
    if (fd == -1) {
        perror("Error opening value file");
        exit(EXIT_FAILURE);
    }
    char str_value = value ? '1' : '0';
    write(fd, &str_value, 1);
    close(fd);
}

int main() {
    int leds[LED_NUM] = {17, 18, 19, 20};  // 假设使用的 GPIO 引脚编号
    int i, j;
    
    for (i = 0; i < LED_NUM; i++) {
        // 导出 GPIO
        int export_fd = open(SYSFS_GPIO_EXPORT, O_WRONLY);
        if (export_fd == -1) {
            perror("Error opening export file");
            return EXIT_FAILURE;
        }
        char gpio_str[3];
        sprintf(gpio_str, "%d", leds[i]);
        write(export_fd, gpio_str, strlen(gpio_str));
        close(export_fd);
        
        // 设置 GPIO 方向为输出
        set_gpio_direction(leds[i], "out");
    }
    
    while (1) {
        // 流水灯效果
        for (i = 0; i < LED_NUM; i++) {
            set_gpio_value(leds[i], 1);
            usleep(200000);
            set_gpio_value(leds[i], 0);
        }
        
        // 呼吸灯效果
        for (j = 0; j < 100; j++) {
            for (i = 0; i < LED_NUM; i++) {
                set_gpio_value(leds[i], 1);
                usleep(j * j);
            }
            for (i = 0; i < LED_NUM; i++) {
                set_gpio_value(leds[i], 0);
                usleep(j * j);
            }
        }
    }
    
    return 0;
}

```

上述代码示例仅为参考，实际使用时需要根据你的硬件和系统配置进行适当修改和测试。

