# 1. 简介

- 本文是基于RK3588平台，SDK版本：RK3588_ANDROID12.0    RK628D调试总结。

- 视频桥接芯片：RK628D

- 驱动代码："kernel-5.10\drivers\misc\rk628"(驱动用的是rk628-for-all-v21版本)

- 本次调试的方案功能：从SOC出来的HDMITX通过RK628D转成双路LVDS信号接LVDS屏幕。

# 2. 视频桥接芯片RK628D调试

## 2.1 RK628驱动介绍

RK628 分为 Display 通路和 HDMI IN 通路，SDK 版本 Display 通路基于DRM框架，HDMI IN 通路基于

V4L2框架。

RK628-For-All 版本驱动一样也分为Display 通路和 HDMI IN 通路，Display 通路的驱动于drivers/misc/rk628/

下，HDMI IN 通路的驱动于drivers/media/i2c/rk628/下。本文采用RK628-For-All 版本Display 通路：MISC

## 2.2 调试总览，调试步骤分析

- 步骤 ①  移植RK628D_For_All_V21的驱动代码

- 步骤 ②  dts配置

- 步骤 ③ 编译，烧写。

 ## 2.3 调试过程

- **步骤 ①  ：移植RK628D_For_All_V21的驱动代码**


 

**1.联系RK业务拿到最新的RK628-for-all版本代码**。

本文是基于RK628-for-all-v21版本。要移植RK628D_For_All的驱动代码。

采取手动打补丁的方式移植：因为自动打补丁会因为SDK版本差异，代码不一致导致报错。


 

**2.rockchip_defconfig，Kconfig，Makefile配置**

**rockchip_defconfig配置**：关闭SDK系统自带的rk628d配置，开启rk628-for-all版本的配置：

CONFIG_DRM=y(系统默认是打开)

CONFIG_RK628_MISC=y

CONFIG_ROCKCHIP_THUNDER_BOOT_RK628=y

(下面两项在"kernel-5.10\drivers\misc\rk628\Kconfig"已经默认设置为y了，可以不用在rockchip_defconfig中再配置)

**MISC配置如下** ：

将rk628驱动添加进编译规则。

Kconfig配置 ：添加 source "drivers/misc/rk628/Kconfig"

Kconfig路径 ："kernel-5.10\drivers\misc\Kconfig"

Makefile配置 ：添加 obj-y                += rk628/

Makefile路径 ："kernel-5.10\drivers\misc\Kconfig"

**rk628-for-all版本驱动配置如下：**

Kconfig配置 ：添加 config RK628_MISC 和 config ROCKCHIP_THUNDER_BOOT_RK628说明

Kconfig路径 ："kernel-5.10\drivers\misc\rk628\Kconfig"

Makefile配置 ：添加RK628_MISC驱动和obj-$(CONFIG_DRM) += rk628_hdmitx.o

Makefile路径 ："kernel-5.10\drivers\misc\rk628\Makefile"


 

3.**驱动手动打补丁：**

①  将rk628文件夹复制到"kernel-5.10\drivers\misc\rk628"

② kernel-5.10\drivers\gpu   hdmi强制输出固定分辨率  绕过读edid流程

    kernel-5.10\drivers\i2c     提前i2c设备的注册 以加快rk628的初始化  

    kernel-5.10\drivers\base  增加宏主要是为了实现regmap文件结点可以写628寄存器

    kernel-5.10\drivers\pwm  提前pwm设备的注册 以加快rk628的初始化

    kernel-5.10\drivers\video 提前backlight设备的注册 以加快rk628的初始化

   注意： drivers\gpu\drm\bridge\synopsys\dw-hdmi-qp.c    此c文件的第一组分辨率改成你要固定的分辨率 1920*1080

- **步骤 ②  dts配置**

1. rk628-for-all的dts配置

```bash

&i2c6 {

    //clock-frequency = <400>;

    pinctrl-names = "default";

    pinctrl-0 = <&i2c6m0_xfer>;

    status = "okay";

    rk628: rk628@50 {

        compatible = "rockchip,rk628";

        reg = <0x50>;

        interrupt-parent = <&gpio4>;

        interrupts = <22 IRQ_TYPE_LEVEL_HIGH>;

        //pinctrl-names = "default";

        //pinctrl-0 = <&rk628power>;

        enable-gpios = <&gpio4 RK_PD5 GPIO_ACTIVE_HIGH>;

        reset-gpios = <&gpio2 RK_PB6 GPIO_ACTIVE_LOW>;

        //panel-enable-gpios = <&gpio2 RK_PC1 GPIO_ACTIVE_HIGH>;

        panel-backlight = <&backlight_lvds>;

        status = "okay";

        rk628,hdmi-in;

        rk628-lvds{

            /* "jeida_18","vesa_24","vesa_18" */

            bus-format = "vesa_24";

            //bus-format = "jeida_18";

            /* "dual_link_odd_even_pixels"

             * "dual_link_even_odd_pixels"

             * "dual_link_left_right_pixels"

             * "dual_link_right_left_pixels"

            */

            link-type = "dual_link_even_odd_pixels";

            //link-type = "dual_link_odd_even_pixels";

            status = "okay";

        };

        display-timings {

            src-timing {

                clock-frequency = <148500000>;

                hactive = <1920>;

                vactive = <1080>;

                hback-porch = <148>;

                hfront-porch = <88>;

                vback-porch = <6>;

                vfront-porch = <4>;

                hsync-len = <44>;

                vsync-len = <5>;

                hsync-active = <0>;

                vsync-active = <0>;

                de-active = <0>;

                pixelclk-active = <0>;

            };

            dst-timing {

                clock-frequency = <148500000>;

                hactive = <1920>;

                vactive = <1080>;

                hback-porch = <148>;

                hfront-porch = <88>;

                vback-porch = <6>;

                vfront-porch = <4>;

                hsync-len = <44>;

                vsync-len = <5>;

                hsync-active = <0>;

                vsync-active = <0>;

                de-active = <0>;

                pixelclk-active = <0>;

            };

        };

    };

};

```

2. hdmi的dts配置：

```bash

&hdmi0 {

    status = "okay";

};

&hdmi0_in_vp0{

status = "okay";

};

&hdptxphy_hdmi0 {

    status = "okay";

 };

&dsi0{

status = "disabled";

};

&dsi1 {

    status = "disabled";

};

```

## 2.4 调试命令，方法

命令：

```bash

1. cat sys/kernel/debug/dri/0/summary

2. dmesg | grep rk628

3. cat /sys/kernel/debug/gpio 查看gpio占用状态

4. dmesg | grep src 查看src(cpu输出的hdmi信号)状态

5. dmesg | grep “rxphy power”  查看rxphy power是否上电

6. dmesg | grep vop查看显示信息或者dmesg | grep drm

7. dmesg | grep stable查看628clock是否起来。

8. cat sys/kernel/debug/rk628/summary

```

# 3. 调试成功
​​
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3d15f1384b54e50b5d85e0567b7fc7f.jpeg)
 ## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)