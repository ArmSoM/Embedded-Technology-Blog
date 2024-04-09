# 1. Bootloader

1. Bootloader是在操作系统运行之前的一段小程序。
2. 通过这段小程序，我们可以初始化硬件设备，建立内存空间的映射表，从而建立适当的系统软硬件环境，为最终调用操作系统内核做好准备。

## 1.1. bootloader种类

| Bootloader   | 描述  | x86  | ARM | PowerPC |
| ------------ |------ | --- | ------ | ------ |
| LILO        | Linux磁盘引导程序 |  是 | 否 | 否 |
| GRUB        | GNU 的LILO代替程序 |  是 | 否 | 否 |
| Loadlin     | 从DOS引导 Linux |  是 | 否 | 否 |
| ROLO        | 从ROM引导Linux而不需要BIOS |  是 | 否 | 否 |
| Etherboot   | 通过以太网卡启动linux系统的固件 |  是 | 否 | 否 |
| LinuxBIOS   | 完全代替BUIS的Linux引导程序 |  否 | 是 | 否 |
| BLOB        | LART等硬件平台的引导程序 |  是 | 否 | 否 |
| U-boot      | 通用引导程序 |  是 | 是 | 是 |
| RedBoot     | 基于eCos的引导程序 |  是 | 是 | 是 |

# 2. uboot

U-Boot, 全称 Universal Boot Loader，是遵循GPL条款的开放源码项目，是一套在GNU通用公共许可证之下发布的自由软件。

# 3. Uboot对硬件平台的支持

U-Boot是一个主要用于嵌入式系统的应道加载程序

可以支持多种不同的计算机系统结构，包括：PPC，ARM，AVR32，MIPS，x86，68k，Nios与MicroBlaze等诸多常用系统的处理器。

# 4. Uboot对os的支持

在操作系统方面，U-Boot不仅支持嵌入式linux系统的引导。
    - 目前支持的目标操作系统有OpenBSD、NetBSD、FreeBSD、4.4BSD、Linux、SVR4、Esix、Solaris、Irix、SCO、Dell、VxWorks、LynxOS

# 5. 嵌入式系统启动流程

1） 设备上电之后，先执行iROM中的出厂代码（启动设备选择）；
2） 执行U-Boot
3)  启动Linux内核；
4） 挂载文件系统，如：rootfs；
5） 执行应用程序linuxrc。

# 6. 存储镜像有哪些外设？

- 1. nand flash 
- 2. emmc/SD
- 3. qspi flash
- 4. 没有外设存储
    - 网络tftp、fastboot(usb)

# 7. 如何拷贝镜像到开发板？

- 1. 网络 tftp
- 2. 串口 loadb + kermit
- 3. usb Fastboot
- 4. JLink/Jtag
- 5. tf卡 fatload