# 1. 简介

- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)
- 本⽂主要介绍在Rockchip平台配置spi接口并且使用的方法
- 开发板：ArmSoM-W3
- Kernel：5.10.160
- OS：Debian11



# 2. SPI接口概述

SPI（Serial Peripheral Interface），即串行外围设备接口，是一种同步的，全双工的，多设备的，多主机的通信协议，用于连接外围设备，如ADC、DAC、数据存储器、定时器、接受器等。



## 2.1 spi接口的结构

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ SPI篇 ]--SPI的开发指南\ArmSom-W3_SPI_structure.png)

1. 主机：主机是SPI总线的控制者，它负责控制数据传输的方向和传输速度。
2. 从机：从机是SPI总线的被控制者，它根据主机发出的指令，发出或接收数据。
3. MOSI（Master Out Slave In）：主机输出从机输入，用于传输从主机到从机的数据。
4. MISO（Master In Slave Out）：主机输入从机输出，用于传输从从机到主机的数据。
5. CK（Serial Clock）：时钟线，用于同步主机和从机之间的数据传输。
6. CS（Chip Select）：片选线，用于控制主机和从机之间的数据传输。

此外，SPI接口还有其他可选项，如中断线（INT）、复位线（RESET）等。



## 2.2 SPI接口的工作原理

在SPI接口通信过程中，主机发出一个片选信号，然后在时钟信号的控制下，主机发出一个字节的数据，从机接收到数据之后，也会发出一个字节的数据，主机接收到数据之后，发出一个片选信号，结束一次通信。

SPI接口有两种工作模式：主模式和从模式。主模式下，主机控制从机，从机接收主机发出的指令。从模式下，从机可以接收主机发出的指令，并向主机发送数据。



# 3.SPI硬件接口

RK3588旗舰芯片上可使用的spi接口有5组，ArmSoM SOM-3588-LGA核心板采用LGA 506引脚封装方式将spi资源全部引出，ArmSoM-W3开发板上40PIN引脚中有两组spi接口：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ SPI篇 ]--SPI的开发指南\ArmSom-W3_SPI_40PIN.png)



# 4.spi设备配置

spi驱动相关代码路径：

```
drivers/spi/spi.c spi驱动框架
drivers/spi/spi-rockchip.c rk spi各接口实现
drivers/spi/spidev.c 创建spi设备节点，用户态使用。
drivers/spi/spi-rockchip-test.c spi测试驱动，需要自己手动添加到Makefile编译
Documentation/spi/spidev_test.c 用户态spi测试工具
```



## 4.1 配置 DTS 节点

```
&spi1 {
	status = "okay";
	//assigned-clock-rates = <200000000>; //默认不用配置，SPI 设备工作时钟
	max-freq = <48000000>; /* spi internal clk, don't modify */
	//dma-names = "tx","rx"; //使能DMA模式
	//rx-sample-delay-ns = <10>; //默认不用配置，读采样延时
	spi_dev@0 {
		compatible = "rockchip,spidev";
		reg = <0>;
		spi-max-frequency = <12000000>;
		spi-lsb-first; //IO 先传输 lsb		
	};
};
```

- status:如果要启用 SPI，则设为 okay，如不启用，设为 disable。

- spi_dev@0:本例子使用 CS0，故此处设为 0，如果使用 CS1，则设为 1。

- compatible:这里的属性必须与驱动中的结构体：of_device_id 中的成员 compatible 保持一致。

- reg:此处与 spi_dev@0 保持一致，这里设为0。

- spi-max-frequency：此处设置 spi 使用的最高频率，ITX-3588J 最高支持 50000000。




这里使用的驱动是drivers/spi/spidev.c，驱动设备加载注册成功后，会出现类似这个名字的设备：/dev/spidev1.0。

一些常见的SPI驱动API接口：

**Linux SPI API**:

- `spi_register_driver()`：注册SPI设备驱动程序。
- `spi_unregister_driver()`：注销SPI设备驱动程序。
- `spi_setup()`：设置SPI总线和设备的参数。
- `spi_sync()`：同步方式进行SPI数据传输。
- `spi_message_init()`：初始化SPI消息结构。
- `spi_message_add_tail()`：向SPI消息添加传输操作。
- `spi_sync()`：同步方式进行SPI数据传输。
- `spi_transfer()`：进行SPI数据传输。

SPI驱动中常用的ioctl请求值，这些请求值用于设置和读取SPI设备的各种参数，包括通信模式、字长、数据模式和通信速率等。这些请求值通常用于Linux的SPI驱动编程。以下是一些常用的SPI ioctl请求值：

- **SPI_IOC_RD_MODE**: 读取SPI设备的通信模式。
- **SPI_IOC_WR_MODE**: 设置SPI设备的通信模式。
- **SPI_IOC_RD_MODE32**: 读取SPI设备的32位通信模式。
- **SPI_IOC_WR_MODE32**: 设置SPI设备的32位通信模式。
- **SPI_IOC_RD_LSB_FIRST**: 读取SPI设备的LSB（Least Significant Bit）优先模式。
- **SPI_IOC_WR_LSB_FIRST**: 设置SPI设备的LSB优先模式。
- **SPI_IOC_RD_BITS_PER_WORD**: 读取SPI设备的字长。
- **SPI_IOC_WR_BITS_PER_WORD**: 设置SPI设备的字长。
- **SPI_IOC_RD_MAX_SPEED_HZ**: 读取SPI设备的最大通信速率。
- **SPI_IOC_WR_MAX_SPEED_HZ**: 设置SPI设备的最大通信速率。
- **SPI_IOC_MESSAGE(N)**: 一次进行N次双向或多次读写操作。



## 4.2 SPI测试实验

将上述设备树以及驱动修改之后更新板卡的内核，通过SPI设备文件来判断spi驱动是否加载成功

这里测试使用的是spi1这一组spi相关接口，检查spi设备：

```
root@linaro-alip:~# ls /dev/spi*
/dev/spidev1.0	
root@linaro-alip:~# 
```

测试SPI接口，使用/dev/spidev1.0设备节点，实现短接MOSI和MISO线路以自发自收数据，同时在未短接时报告错误



### 4.2.1 硬件连接

将SPI1的 MIOS与MOSI引脚(板卡上的7和8)短接即可，如下图所示

<img src="C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ SPI篇 ]--SPI的开发指南\ArmSom-W3_SPI_connect.png" style="zoom: 50%;" />



## 4.2.2 测试程序

用户应用层使用spidev驱动的步骤如下：

1. 打开SPI设备文件：用户可以通过打开/dev/spidevX.Y文件来访问SPI设备，其中X是SPI控制器的编号，Y是SPI设备的编号。

2. 配置SPI参数：用户可以使用ioctl命令SPI_IOC_WR_MODE、SPI_IOC_WR_BITS_PER_WORD和SPI_IOC_WR_MAX_SPEED_HZ来设置SPI模式、数据位数和时钟速度等参数。


3. 发送和接收数据：用户可以使用read和write系统调用来发送和接收SPI数据。写入的数据将被传输到SPI设备，而从设备读取的数据将被存储在用户提供的缓冲区中。

4. 关闭SPI设备文件：当不再需要与SPI设备通信时，用户应该关闭SPI设备文件。

总结起来，spidev驱动提供了一种简单而灵活的方式来与SPI设备进行通信，使得用户可以轻松地在Linux系统上开发和控制SPI设备。

armsom团队 的测试程序spi_test.c如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>
#include <linux/spi/spidev.h>

#define SPI_DEV_PATH "/dev/spidev1.0"

int fd;
static unsigned mode = SPI_MODE_0;
static uint8_t bits = 8;
static uint32_t speed = 1000000; // 设置SPI速度为1MHz
static uint16_t delay;

void transfer(int fd, uint8_t const *tx, uint8_t *rx, size_t len)
{
    int ret;
    struct spi_ioc_transfer tr = {
        .tx_buf = (unsigned long)tx,
        .rx_buf = (unsigned long)rx,
        .len = len,
        .delay_usecs = delay,
        .speed_hz = speed,
        .bits_per_word = bits,
        .cs_change = 0, // 设置为1以在每次传输前切换片选，这里不切换片选
    };

    ret = ioctl(fd, SPI_IOC_MESSAGE(1), &tr);

    if (ret < 1) {
        perror("SPI transfer failed");
    }
}

void spi_init(void)
{
    int ret;
    // 打开 SPI 设备
    fd = open(SPI_DEV_PATH, O_RDWR);
    if (fd < 0) {
        perror("Can't open SPI device");
        exit(1);
    }

    // 设置 SPI 工作模式
    ret = ioctl(fd, SPI_IOC_WR_MODE, &mode);
    if (ret == -1) {
        perror("Can't set SPI mode");
        exit(1);
    }

    // 设置位数
    ret = ioctl(fd, SPI_IOC_WR_BITS_PER_WORD, &bits);
    if (ret == -1) {
        perror("Can't set bits per word");
        exit(1);
    }

    // 设置SPI速度
    ret = ioctl(fd, SPI_IOC_WR_MAX_SPEED_HZ, &speed);
    if (ret == -1) {
        perror("Can't set max speed");
        exit(1);
    }

    // 打印设置
    printf("SPI mode: 0x%x\n", mode);
    printf("Bits per word: %d\n", bits);
    printf("Max speed: %d Hz\n", speed);
}

int main(int argc, char *argv[])
{
    if (argc != 2) {
        printf("Usage: %s <string_to_send>\n", argv[0]);
        return 1;
    }

    char *tx_buffer = argv[1]; // 获取要发送的字符串作为命令行参数

    // 初始化SPI接口
    spi_init();

    // 设置要接收数据的缓冲区
    unsigned char rx_buffer[strlen(tx_buffer) + 1];

    // 执行SPI数据传输
    transfer(fd, tx_buffer, rx_buffer, strlen(tx_buffer));

    // 打印发送和接收的数据
    printf("Sent: %s\n", tx_buffer);
    printf("Received: %s\n", rx_buffer);

    // 关闭SPI设备
    close(fd);

    return 0;
}
```



### 4.2.3 编译&运行

开发板上下载gcc编译器：

```
apt upgrade
apt update
apt install gcc
```

编译：

```
gcc spi_test.c -o spi
```

短接与不短接的运行情况如下：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ SPI篇 ]--SPI的开发指南\ArmSom-W3_SPI_run.png)

