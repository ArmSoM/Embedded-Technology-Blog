Platform: RK3588
OS: Debian11
Kernel: v5.10.160
Module ：EC200A-CN 国内版

# Linux USB 驱动架构

USB 主机控制器驱动在分层结构的最底层，直接与硬件交互。USB 核心是整个 USB 主机驱动的核心，用于管理 USB 总线、USB 总线设备和 USB 总线带宽；它为 USB 设备驱动程序提供接口，应用程序可以通过这些接口访问 USB 系统文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/2d22a285627a4a9fb41f5be1c08a23dd.png)


# 修改内核配置

## usb转串口

模块加载 USB 转串口 option 驱动程序后，在/dev 目录下创建 ttyUSB0、ttyUSB1 和 ttyUSB2 等设备
文件。需要使能内核选项如下：

> CONFIG_USB_SERIAL 
> CONFIG_USB_SERIAL_WWAN 
> CONFIG_USB_SERIAL_OPTION

## GobiNet 驱动

当模块成功加载 GobiNet 驱动程序后，将创建一个网络设备和一个 QMI 设备节点。网络设备名称为
“usbX”，QMI 设备节点名称为“/dev/qcqmiX”。网络设备用于数据传输，QMI 设备节点用于 QMI 消息
交互。需先启用以下配置项：

> CONFIG_USB_NET_DRIVERS
>  CONFIG_USB_USBNET

## QMI_WWAN 驱动

当模块成功加载 QMI_WWA 驱动程序后，会创建一个网络设备和一个 QMI 设备节点。网络设备名称为“wwanX”，QMI 设备节点名称为“/dev/cdc-wdmX”。网络设备用于数据传输，QMI 设备节点用于 QMI消息交互。需先启用以下配置项：

> CONFIG_USB_NET_QMI_WWAN 
> CONFIG_USB_WDM

## usb网卡驱动

USB 接口类驱动程序，在上游Linux 版本中可用。当模块通过 USB 接口连接到 Linux PC 时，会自动加载驱动程序

> USB_USBNET=y 
> USB_NET_CDCETHER=y        #用ECM  使能此项
> USB_NET_RNDIS_HOST=y      #用RNDIS 使能此项

## 支持 PPP功能

启用以下内核配置项来支持 PPP

> CONFIG_PPP 
> CONFIG_PPP_ASYNC 
> CONFIG_PPP_SYNC_TTY 
> CONFIG_PPP_DEFLATE

# 配置内核

> cd SDK/kernel 
> make ARCH=arm64 “defconfig”文件(默认是rockchip_linux_defconfig) 
> make ARCH=arm64 menuconfig
> cp .config arch/arm64/configs/“defconfig”文件

进入图形化配置界面后可以使用“/”搜索对应配置
选择<*>将驱动程序编译到内核映像

 ## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)