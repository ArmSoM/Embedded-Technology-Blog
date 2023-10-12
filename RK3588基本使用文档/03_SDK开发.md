# 一、SDK 开发

本节向用户介绍如何对 RK3588 Linux SDK 进行开发，包括 u-boot 开发、Linux 内核开发、buildroot 根文件系统开发、Debian应用开发等；armsom-W3开发板的源码可以在https://github.com/ArmSoM/armsom-w3-bsp获取，用户可以基于本 SDK 进行二次开发、软件定制，以适配自己的 Linux 产品；基于本 SDK，可以有效实现系统定制和应用移植开发，帮助用户快速开发、提高开发效率！

# 二、SDK 板级配置文件

SDK 板级配置文件中提供了一些必要的配置信息。对于 RK3588 平台，其板级配置文件位于<SDK>/device/rockchip/rk3588/目录，如下所示：

<img src="C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_1.png" style="zoom:150%;" />

该目录下有多个 BoardConfig-xxxx.mk 文件，这些.mk 文件都是板级配置文件，每一个.mk 文件都对应一个开发板的资源，其中
BoardConfig-rk3588-armsom-w3.mk 就是我们的armsom-w3开发板所使用的板级配置文件。
我们在 SDK 根目录下执行“./build.sh lunch”或者第一次使用"./build.sh"编译时所列举出来的文件就是从<SDK>/device/rockchip/rk3588/目录来的，如下所示： 
<img src="C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_2.png" style="zoom:150%;" />

这里需要选择**“2”**

```
Which would you like? [0]: 2
switching to board: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/device/rockchip/rk3588/BoardConfig-rk3588-armsom-w3.mk
```

这些.mk 文件其实是一个 sh 脚本文件，打开 BoardConfig-rk3588-armsom-w3.mk 配置文件，来看一下里面的内容：

```
#!/bin/bash

# Target arch
export RK_KERNEL_ARCH=arm64
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=rk3588_armsom_w3
# Uboot image format type: fit(flattened image tree)
export RK_UBOOT_FORMAT_TYPE=fit
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=armsom_w3_defconfig
# Kernel defconfig fragment
export RK_KERNEL_DEFCONFIG_FRAGMENT=rk3588_linux.config
# Kernel dts
export RK_KERNEL_DTS=rk3588-armsom-w3
# boot image type
export RK_BOOT_IMG=boot.img
# kernel image path
export RK_KERNEL_IMG=kernel/arch/arm64/boot/Image
# kernel image format type: fit(flattened image tree)
export RK_KERNEL_FIT_ITS=boot.its
# parameter for GPT table
export RK_PARAMETER=parameter.txt
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3588
# Recovery config
export RK_CFG_RECOVERY=rockchip_rk3588_recovery
# Recovery image format type: fit(flattened image tree)
export RK_RECOVERY_FIT_ITS=boot4recovery.its
# Pcba config
export RK_CFG_PCBA=rockchip_rk3588_pcba
# target chip
export RK_CHIP=rk3588
# Set rootfs type, including ext2 ext4 squashfs
export RK_ROOTFS_TYPE=ext4
# debian version (debian10: buster, debian11: bullseye)
export RK_DEBIAN_VERSION=bullseye
# yocto machine
export RK_YOCTO_MACHINE=rockchip-rk3588-evb
#misc image
export RK_MISC=wipe_all-misc.img
# Define package-file
export RK_PACKAGE_FILE=rk3588-package-file
# Define WiFi BT chip
# Compatible with Realtek and AP6XXX WiFi : RK_WIFIBT_CHIP=ALL_AP
# Compatible with Realtek and CYWXXX WiFi : RK_WIFIBT_CHIP=ALL_CY
# Single WiFi configuration: AP6256 or CYW43455: RK_WIFIBT_CHIP=AP6256
export RK_WIFIBT_CHIP=ALL_AP
# Define BT ttySX
export RK_WIFIBT_TTY=ttyS6
```

- RK_ARCH：用于指定目标架构，rk3588 对应 arm64；
- RK_UBOOT_DEFCONFIG：用于指定 U-Boot 的 defconfig 配置文件；/u-boot/configs/rk3588_armsom_w3。
- RK_UBOOT_FORMAT_TYPE：用于指定 uboot.img 镜像的格式，rk3588 平台默认使用的是 fit（flattened image tree）格式镜像；
- RK_KERNEL_DEFCONFIG：用于指定 Linux Kernel（内核）的 defconfig 配置文件；/kernel/arch/arm64/configs/rk3588_armsom_w3。
- RK_KERNEL_DEFCONFIG_FRAGMENT：用于指定 Linux 内核的 defconfig fragment，对于 rk3588 来说是空置；
- RK_KERNEL_DTS：用于指定内核设备树文件；rk3588-armsom-w3.dts。
- RK_BOOT_IMG：设置为 boot.img 即可；
- RK_KERNEL_IMG：用于指定内核镜像的路径；kernel/arch/arm64/boot/Image。
- RK_KERNEL_FIT_ITS：rk3588 平台 Linux 系统使用的启动镜像 boot.img 也是 FIT 格式
- 镜像，FIT 使用 its（image source file）文件来描述 image 的信息，RK_KERNEL_FIT_ITS 用于
- 指定这个 its 文件，这个 its 文件必须要存放在<SDK>/device/rockchip/rk3588/目录下；boot.its。
- RK_PARAMETER：用于指定分区表文件，分区表文件里面介绍了bootloader、uboot、kernel等分区的大小及位置
- RK_CFG_BUILDROOT：用于指定 buildroot 根文件系统（普通模式）的 defconfig 配置文件；rockchip_rk3588。
- RK_CFG_RECOVERY：用于指定recovery模式下根文件系统（recovery模式下使用ramdisk
- 根文件系统）的 defconfig 配置文件；rockchip_rk3588_recovery_defconfig。
- RK_RECOVERY_FIT_ITS：用于指定 recovery.img 镜像对应的 its 文件。recovery.img 也
- 是 FIT 格式镜像，需要使用 its 文件来描述 image 的信息；boot4recovery.its。
- RK_CFG_PCBA：用于指定 PCBA 的 defconfig 配置文件；
- RK_ROOTFS_TYPE：用于指定根文件系统的类型，譬如 ext2、ext4；
- RK_DEBIAN_VERSION：用于指定 Debian 的版本，debian10: buster, debian11: bullseye，默认使用的是 Debian 11，不要去改动它；
- RK_YOCTO_MACHINE：编译 yocto 时，用于指定 machine；
- RK_MISC：用于指定 misc 镜像。编译完 SDK 后，生成的<SDK>/rockdev/misc.img 镜像其
- 实就是 RK_MISC 所指定的这个文件，只不过是进行了重命名而已；RK_MISC 所指定的 misc
- 镜像必须要存放在<SDK>/device/rockchip/rockimg 目录下；
- RK_WIFIBT_CHIP:有三类wifi/bt平台，RTK、AP、CY,根据芯片选择去对应的驱动，比如AP6256，那就是RK_WIFIBT_CHIP=AP6256，选择ALL_AP会将编译所有AP系列芯片的驱动
- RK_WIFIBT_TTY:蓝牙使用的串口

用户可以在<SDK>/device/rockchip/rk3588目录下添加自己的板级配置文件，根据编译脚本实际情况对配置文件中的变量进行修改、或添加新的变量

#  三、U-Boot 开发

U-Boot 源码在<SDK>/u-boot 目录，是 RK 从 U-Boot 官⽅的 v2017.09 正式版本中切出来进⾏开发的版本

## 3.1 U-Boot 的设备树

U-Boot 中，armsom-w3的设备树文件是u-boot/arch/arm/dts/rk3588-armsom-w3.dts，该设备树文件包含了 rk3588.dtsi 和 rk3588-u-boot.dts
原生的 U-Boot 只支持 U-Boot 自己的 DTB，RK 平台在原生 U-Boot 基础上增加了 kernel DTB 机制的支持，即 U-Boot 会使用 kernel DTB 去初始化外设。这样设计的目的主要是为了兼容外设板级差异，譬如 power、clock、display 等。U-Boot 设备树负责初始化存储、调试串口等基础外设；而 kernel 设备树初始化存储、调试串口之外的外设，譬如 LCD 显示、千兆网等。执行 U-Boot 代码时先用 U-Boot 的设备树完成存储、调试串口的初始化操作，然后从存储上加载 kernel 的设备树并转而使用这份设备树继续初始化其余外设。
所以用户一般不需要去修改 U-Boot 的设备树文件（除非更换调试串口）。

## 3.2 U-Boot 编译


在SDK源码目录下执行如下命令编译 U-Boot：

```
lhd@ydtx:~/project_code/3588/3588_linux5.10_v1.0.5$ ./build.sh uboot
processing option: uboot
Using prebuilt GCC toolchain: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
============Start building uboot============
TARGET_UBOOT_CONFIG=rk3588_armsom_w3
=========================================
## make rk3588_armsom_w3_defconfig -j24
#
# configuration written to .config
#
............
SEC=1
pack u-boot.itb okay! Input: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/rkbin/RKTRUST/RK3588TRUST.ini

FIT description: FIT Image with ATF/OP-TEE/U-Boot/MCU
Created:         Tue Sep 26 15:51:40 2023
 Image 0 (uboot)
  Description:  U-Boot
  Created:      Tue Sep 26 15:51:40 2023
  Type:         Standalone Program
  Compression:  uncompressed
  Data Size:    1290088 Bytes = 1259.85 KiB = 1.23 MiB
  Architecture: AArch64
  Load Address: 0x00200000
  Entry Point:  unavailable
  Hash algo:    sha256
  Hash value:   fab278a4d10ea177443167fbf78cd9e1372b18cbe143c605d9ad70a637201cb4
 Image 1 (atf-1)
  Description:  ARM Trusted Firmware
  Created:      Tue Sep 26 15:51:40 2023
  Type:         Firmware
  Compression:  uncompressed
  Data Size:    194404 Bytes = 189.85 KiB = 0.19 MiB
  Architecture: AArch64
  Load Address: 0x00040000
  Hash algo:    sha256
  Hash value:   909ea141064f20ffbe6d0c7afbe71e7cfdcd0085014c48b1d8df5cff882e89a7
 Image 2 (atf-2)
  Description:  ARM Trusted Firmware
  Created:      Tue Sep 26 15:51:40 2023
  Type:         Firmware
  Compression:  uncompressed
  Data Size:    24576 Bytes = 24.00 KiB = 0.02 MiB
  Architecture: AArch64
  Load Address: 0x000f0000
  Hash algo:    sha256
  Hash value:   6a970ae6b4d9c8d0dd5f46ea5cb3c27de910aa12d054299d451d5aae6f62bab4
 Image 3 (atf-3)
  Description:  ARM Trusted Firmware
  Created:      Tue Sep 26 15:51:40 2023
  Type:         Firmware
  Compression:  uncompressed
  Data Size:    20480 Bytes = 20.00 KiB = 0.02 MiB
  Architecture: AArch64
  Load Address: 0xff100000
  Hash algo:    sha256
  Hash value:   3ea8cf0d7eeba0b72bed08ca20e02bc50a50def876fb582b0b6649a7f843df38
 Image 4 (optee)
  Description:  OP-TEE
  Created:      Tue Sep 26 15:51:40 2023
  Type:         Firmware
  Compression:  uncompressed
  Data Size:    461200 Bytes = 450.39 KiB = 0.44 MiB
  Architecture: AArch64
  Load Address: 0x08400000
  Hash algo:    sha256
  Hash value:   fde08608450331a80c98b86e21933df13ad84dd7647af5354f38381d9b42ab12
 Image 5 (fdt)
  Description:  U-Boot dtb
  Created:      Tue Sep 26 15:51:40 2023
  Type:         Flat Device Tree
  Compression:  uncompressed
  Data Size:    10058 Bytes = 9.82 KiB = 0.01 MiB
  Architecture: AArch64
  Hash algo:    sha256
  Hash value:   e388acd7c02c1af89b73326a103f7b03c8716d2071442ef45fe1618ff8283205
 Default Configuration: 'conf'
 Configuration 0 (conf)
  Description:  rk3588-armsom-w3
  Kernel:       unavailable
  Firmware:     atf-1
  FDT:          fdt
  Loadables:    uboot
                atf-2
                atf-3
                optee
********boot_merger ver 1.2********
Info:Pack loader ok.
pack loader okay! Input: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/rkbin/RKBOOT/RK3588MINIALL.ini
/home/lhd/project_code/3588/3588_linux5.10_v1.0.5/u-boot

Image(no-signed, version=0): uboot.img (FIT with uboot, trust...) is ready
Image(no-signed): rk3588_spl_loader_v1.09.111.bin (with spl, ddr...) is ready
pack uboot.img okay! Input: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/rkbin/RKTRUST/RK3588TRUST.ini

Platform RK3588 is build OK, with new .config(make rk3588_armsom_w3_defconfig -j24)
/home/lhd/project_code/3588/3588_linux5.10_v1.0.5/prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
Tue Sep 26 15:51:40 CST 2023
Running build_uboot succeeded.
```

编译完成后将会生成 uboot.img 和rk3588_spl_loader_v1.09.111.bin 两个镜像文件

### 3.3 uboot.img 镜像

uboot.img 是由多个镜像合并而成，包括 u-boot 镜像、u-boot dtb 以及 trust 镜像（ARM Trusted Firmware + OP-TEE）。uboot.img 是一种 FIT（flattened image tree）格式镜像，支持任意多个 image 打包和校验。使用 file 命令查看 uboot.img，如下所示：
![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_3.png)

FIT 使用 its（image source file）文件来描述 image 的信息，最后通过 mkimage 工具生成 itb（flattened image tree blob）镜像，那么这个 itb 镜像其实就是 uboot.img 镜像，uboot.img 镜像通常含有多份 itb 镜像，如下所示：

```
uboot.img = uboot.itb * N（N 一般是 2）
```

这种设计也是为了避免如果第一份镜像启动失败、还可以尝试使用第二份镜像启动。
U-Boot 编译成功后，U-Boot 源码目录下会生成很多.bin 镜像以及.dtb 镜像,uboot.img 镜 像 最 终 由  u-boot.dtb、bl31_0x00040000.bin、bl31_0x000f0000.bin、bl31_0xff100000.bin、rk3588_spl_loader_v1.09.111.bin、tee.bin、u-boot-dtb.bin、u-boot-nodtb.bin、u-boot.bin 这些镜像合并而成。uboot.img 是 FIT 格式镜像，使用 its 文件来描述 image 的信息，最终通过<UBoot>/tools/mkimage 工具生成 itb 镜像；its 文件和生成的 itb 文件都在<U-Boot>/fit 目录下，u-boot.its 文件中描述了有哪些镜像会参与合并成 uboot.itb，以及这些镜像的路径等信息：

```
/dts-v1/;

/ {
	description = "FIT Image with ATF/OP-TEE/U-Boot/MCU";
	#address-cells = <1>;

	images {

		uboot {
			description = "U-Boot";
			data = /incbin/("u-boot-nodtb.bin");
			type = "standalone";
			arch = "arm64";
			os = "U-Boot";
			compression = "none";
			load = <0x00200000>;
			hash {
				algo = "sha256";
			};
		};
		atf-1 {
			description = "ARM Trusted Firmware";
			data = /incbin/("./bl31_0x00040000.bin");
			type = "firmware";
			arch = "arm64";
			os = "arm-trusted-firmware";
			compression = "none";
			load = <0x00040000>;
			hash {
				algo = "sha256";
			};
		};
		atf-2 {
			description = "ARM Trusted Firmware";
			data = /incbin/("./bl31_0x000f0000.bin");
			type = "firmware";
			arch = "arm64";
			os = "arm-trusted-firmware";
			compression = "none";
			load = <0x000f0000>;
			hash {
				algo = "sha256";
			};
		};
		atf-3 {
			description = "ARM Trusted Firmware";
			data = /incbin/("./bl31_0xff100000.bin");
			type = "firmware";
			arch = "arm64";
			os = "arm-trusted-firmware";
			compression = "none";
			load = <0xff100000>;
			hash {
				algo = "sha256";
			};
		};
		optee {
			description = "OP-TEE";
			data = /incbin/("tee.bin");
			type = "firmware";
			arch = "arm64";
			os = "op-tee";
			compression = "none";
			
			load = <0x8400000>;
			hash {
				algo = "sha256";
			};
		};
		fdt {
			description = "U-Boot dtb";
			data = /incbin/("./u-boot.dtb");
			type = "flat_dt";
			arch = "arm64";
			compression = "none";
			hash {
				algo = "sha256";
			};
		};
	};

	configurations {
		default = "conf";
		conf {
			description = "rk3588-armsom-w3";
			rollback-index = <0x0>;
			firmware = "atf-1";
			loadables = "uboot", "atf-2", "atf-3", "optee";
			
			fdt = "fdt";
			signature {
				algo = "sha256,rsa2048";
				
				key-name-hint = "dev";
				sign-images = "fdt", "firmware", "loadables";
			};
		};
	};
};
```

its 文件的语法规则与 DTS 是完全相同的，并无差别。

​	对于 ARM Trusted Firmware 以及 OP-TEE，它们是闭源的，RK 只提供了二进制镜像文件，并没提供源码。ARM Trusted Firmware 固件对应<U-Boot>/bl31.elf、OP-TEE 固件对应<UBoot>/tee.bin，编译 U-Boot 源码之前，这两个镜像是不存在的，编译之后才会出现；实际上来自于
<SDK>/rkbin/bin/rk35/rk3588_bl31_v1.32.elf 和<SDK>/rkbin/bin/rk35/rk3588_bl32_v1.12.bin这两个镜像文件。打包 uboot.itb 的过程中，会将<SDK>/rkbin/bin/rk35/rk3588_bl31_v1.32.elf 拷贝到<U-Boot>/bl31.elf，将<SDK>/rkbin/bin/rk35/rk3588_bl32_v1.12.bin 拷贝到<U-Boot>/tee.bin。
bl31.elf 固件并不是直接打包进 uboot.itb，而是将 bl31.elf 分解成多个 bl31_xxx.bin 文件，最终将这些 bl31_xxx.bin 镜像打包进 uboot.itb。

### 3.4 rk3588_spl_loader_v1.09.111.bin 镜像

rk3588_spl_loader_v1.09.111.bin这个镜像文件就是MiniLoaderAll.bin 镜像 ，名字不一样。MiniLoaderAll.bin 是运行在 RK3588 平台 U-Boot 之前的一段 Loader 代码（也就是比 U-Boot 更早阶段的 Loader）。MiniLoaderAll.bin 由两部分构成：TPL(Tiny Program Loader) + SPL(Secondary Program Loader)构成。
	TPL 运行在 SRAM 中（片内内存），由 rk3568 芯片内部所固化的 Maskrom（BootROM）代码引导启动；其作用是负责完成 DRAM 的初始化工作、并启动 SPL；SPL 运行在 DDR，SPL的作用是负责完成系统的 lowlevel 初始化、完成 uboot.img 的加载和引导工作。

### 3.5 镜像启动顺序

	这里讲一下 RK3568 平台镜像的启动顺序。涉及到 Trust，目前 Rockchip 的 64 位 SoC 平台
上使用的是 ARM Trusted Firmware + OP-TEE 的组合来实现 Trust，32 位 SoC 平台上使用的
是 OP-TEE。
	ARM Trusted Firmware 的体系架构里将整个系统分成四种安全等级，分别为：EL0、EL1、EL2、EL3。将整个安全启动的流程阶段定义为：BL1、BL2、BL31、BL32、BL33，其中 ARM Trusted Firmware 自身的源代码里提供了 BL1、BL2、BL31 的功能。Rockchip 平台仅使用了其中的 BL31 的功能；对于 BL1 和 BL2，RK 有自己的一套实现方案。所以在 Rockchip 平台上，我们一般也可以“默认”ARM Trusted Firmware 指的就是 BL31，而 BL32 使用的则是 OP-TEE。

​	如果把上述这种阶段定义映射到 Rockchip 平台各级固件上，对应关系为：Maskrom（RK 芯片内部固化的引导代码，也叫 BootROM，BL1）、MiniLoaderAll.bin（BL2）、Trust（BL31：ARM Trusted Firmware + BL32：OP-TEE）、U-Boot（BL33）。

> 所以 Linux 系统的镜像启动顺序为：
> Maskrom → MiniLoaderAll.bin → uboot.img → boot.img → rootfs.img
> 还可以进行细分：
> Maskrom → TPL(ddr bin) → SPL(miniloader) → Trust(ATF + OP-TEE) → u-boot → kernel → rootfs

这个启动流程通过打印信息就可以分析出来，以下就是armsom-w3 开发板上电启动时的打印信息：

```
DR V1.09 a930779e06 typ 22/11/21-17:50:56 //DDR版本
# 对开发板的 DDR 进行初始化
LPDDR4X, 2112MHz
channel[0] BW=16 Col=10 Bk=8 CS0 Row=16 CS1 Row=16 CS=2 Die BW=16 Size=2048MB
channel[1] BW=16 Col=10 Bk=8 CS0 Row=16 CS1 Row=16 CS=2 Die BW=16 Size=2048MB
channel[2] BW=16 Col=10 Bk=8 CS0 Row=16 CS1 Row=16 CS=2 Die BW=16 Size=2048MB
channel[3] BW=16 Col=10 Bk=8 CS0 Row=16 CS1 Row=16 CS=2 Die BW=16 Size=2048MB
Manufacturer ID:0xff
CH0 RX Vref:29.7%, TX Vref:22.8%,22.8%
CH1 RX Vref:31.7%, TX Vref:21.8%,21.8%
CH2 RX Vref:28.7%, TX Vref:23.8%,22.8%
CH3 RX Vref:30.7%, TX Vref:22.8%,21.8%
change to F1: 528MHz
change to F2: 1068MHz
change to F3: 1560MHz
change to F0: 2112MHz
out
# ddr初始化结束，开始运行miniloader代码
U-Boot SPL board init
U-Boot SPL 2017.09-gc060f28d70-220414 #zyf (Apr 18 2022 - 18:13:34)
Failed to set cpub01
Failed to set cpub23
unknown raw ID phN
unrecognized JEDEC id bytes: 00, 00, 00
Trying to boot from MMC2
MMC: no card present
mmc_init: -123, time 1
spl: mmc init failed with error: -123
Trying to boot from MMC1
Trying fit image at 0x4000 sector
## Verified-boot: 0
## Checking atf-1 0x00040000 ... sha256(909ea14106...) + OK
## Checking uboot 0x00200000 ... sha256(52a313d406...) + OK
## Checking fdt 0x0033af68 ... sha256(e388acd7c0...) + OK
## Checking atf-2 0x000f0000 ... sha256(6a970ae6b4...) + OK
## Checking atf-3 0xff100000 ... sha256(3ea8cf0d7e...) + OK
## Checking optee 0x08400000 ... sha256(fde0860845...) + OK
Jumping to U-Boot(0x00200000) via ARM Trusted Firmware(0x00040000)
Total: 115.946 ms

INFO:    Preloader serial: 2
NOTICE:  BL31: v2.3():v2.3-468-ge529a2760:derrick.huang
NOTICE:  BL31: Built : 09:59:49, Nov 21 2022
INFO:    spec: 0x1
INFO:    ext 32k is not valid
INFO:    ddr: stride-en 4CH
INFO:    GICv3 without legacy support detected.
INFO:    ARM GICv3 driver initialized in EL3
INFO:    valid_cpu_msk=0xff bcore0_rst = 0x0, bcore1_rst = 0x0
INFO:    system boots from cpu-hwid-0
INFO:    idle_st=0x21fff, pd_st=0x11fff9, repair_st=0xfff70001
INFO:    dfs DDR fsp_params[0].freq_mhz= 2112MHz
INFO:    dfs DDR fsp_params[1].freq_mhz= 528MHz
INFO:    dfs DDR fsp_params[2].freq_mhz= 1068MHz
INFO:    dfs DDR fsp_params[3].freq_mhz= 1560MHz
INFO:    BL31: Initialising Exception Handling Framework
INFO:    BL31: Initializing runtime services
INFO:    BL31: Initializing BL32
INFO:    hdmirx_handler: dma not on, ret
I/TC: 
I/TC: OP-TEE version: 3.13.0-652-g4542e1efd #derrick.huang (gcc version 10.2.1 20201103 (GNU Toolchain for the A-profile Architecture 10.2-2020.11 (arm-10.16))) #5 2022年 09月 20日 星期二 09:41:09 CST aarch64
I/TC: Primary CPU initializing
I/TC: Primary CPU switching to normal world boot
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x200000
INFO:    SPSR = 0x3c9

# 进入到 U-Boot，开始执行 U-Boot 代码
U-Boot 2017.09 (Sep 11 2023 - 18:03:37 +0800)

Model: armsom w3
PreSerial: 2, raw, 0xfeb50000
DRAM:  8 GiB
Sysmem: init
Relocation Offset: eda3c000
Relocation fdt: eb9f9c38 - eb9fecd0
CR: M/C/I
Using default environment

Hotkey: ctrl+` //ctrl+ "c"进入uboot命令行
......
Total: 861.58 ms

Starting kernel ...
......
```

**对于整个 U-Boot 来说，如果你的产品没什么特殊的需求，基本不用去动 U-Boot 源码**



# 四、kernel 开发

Linux 内核源码在<SDK>/kernel 目录下：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_4.png)



## 4.1 内核设备树和 defconfig 配置文件

armsom-w3开发板Linux Kernel 所使用的 defconfig 配置文件为<SDK>/kernel/arch/arm64/configs/armsom_w3_defconfig。用户可以根据自己的需求更改该文件、使能或禁用内核模块。Rockchip 平台的所有设备树文件都存放在<SDK>/kernel/arch/arm64/boot/dts/rockchip/目录下。对于 armsom-w3 开发板来说，使用的设备树文件为：rk3588-armsom-w3.dts。
rk3588-armsom-w3.dts 包含有多个.dtsi 设备树，如下所示：

```
#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/leds/common.h>
#include <dt-bindings/pwm/pwm.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/input/rk-input.h>
#include <dt-bindings/display/drm_mipi_dsi.h>
#include <dt-bindings/display/rockchip_vop.h>
#include <dt-bindings/sensor-dev.h>
#include "dt-bindings/usb/pd.h"
#include "rk3588.dtsi"
#include "rk3588-rk806-single.dtsi"
#include "rk3588-linux.dtsi"
#include "rk3588-armsom-w3-display.dtsi"
#include "rk3588-armsom-w3-camera.dtsi"
```

- rk3568.dtsi：该设备树文件是 RK3568 平台级设备树文件，与具体开发板硬件无关，纯SoC 级别的设备树文件，由 RK 提供，开发者无需改动该文件！
- rk3588-rk806-single.dtsi：使用的PMIC电源控制方案,里面描述的是具体的硬件电源节点
- rk3568-linux.dtsi：该设备树包含 Linux 部分特有配置信息
- rk3588-armsom-w3-display.dtsi：关于armsom-w3开发板上面的mipi屏幕配置
- rk3588-armsom-w3-camera.dtsi：关于armsom-w3开发板上面的摄像头配置

rk3588-armsom-w3.dts设备树是提供给用户使用的，用户在编译内核源码时只需编译这个设备树。

对于开发板上面使用到的硬件，可以在设备树文件下面`disabled/okay`，除了修改设备树，还可以对内核进行配置。

在<SDK>/kernel/使用如下命令修改内核的配置：

```
make ARCH=arm64 armsom_w3_defconfig
make ARCH=arm64 menuconfig
make ARCH=arm64 savedefconfig
cp .config arch/arm64/configs/armsom_w3_defconfig
```

执行make ARCH=arm64 menuconfig后进入图形化内核配置界面

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_5.png)

对于需要改动的驱动使用“/”键进去搜索界面
![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_6.png)

输入关键词，比如触摸屏的驱动是gt9xx，那搜索gt9就会索引到配置位置

按空格键选择配置状态：[*]编译进内核 [M]编译成ko文件 [ ]不编译

## 4.2 内核编译

sdk的交叉编译工具链是`prebuilts\gcc\linux-x86\aarch64\gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu\bin\aarch64-none-linux-gnu-gcc`，在SDK源码目录下直接执行`./build.sh kernel`直接编译内核代码,第一次编译时间会有点久，之后再编译就只会编译修改的内核驱动文件，编译完成后，会生成 boot.img 以及 resource.img

```
lhd@ydtx:~/project_code/3588/3588_linux5.10_v1.0.5/kernel$ ls -l boot.img resource.img 
-rw-rw-r-- 1 lhd lhd 32218624  9月 21 18:11 boot.img
-rw-rw-r-- 1 lhd lhd   201216  9月 21 18:11 resource.img
```

关于这两个镜像，这里说明一下

### 4.2.1 boot.img 镜像

RK3588 平台 Linux 系统使用的 boot.img 是一种 FIT 格式镜像，它由多个镜像合并而成，对于 RK3588 平台来说，烧录到开发板 boot 分区的 boot.img 包含了内核镜像 Image、内核 DTB、resource.img 这三部分。可以使用 file 命令查看 boot.img 文件：

```
lhd@ydtx:~/project_code/3588/3588_linux5.10_v1.0.5/kernel$ file boot.img 
boot.img: Device Tree Blob version 17, size=1536, boot CPU=0, string block size=190, DT structure block size=1004
```

“Device Tree Blob”表示该文件是一个 FIT 格式镜像。通过 mk-fitimage.sh 脚本打包生成的 boot.img 才是最终烧录到开发板 boot 分区（Linux 系统）的 boot.img

**如果改动了设备树文件，需要重新编译设备树得到新的 Kernel DTB，然后将它打包进 resource.img、之后再把 resource.img 打包进 boot.img，最后将新的 boot.img 烧录到开发板 boot 分区完成替换；虽然很麻烦，但必须得这么做。**

### 4.2.2 resource.img 镜像

resource.img 是 RK 自己设计的一种镜像，用于存放一些资源，譬如 u-boot logo 图片、内核logo 图片以及设备树镜像 DTB。resource.img 并不是单独烧录到开发板，而是将其打包进boot.img 中，最终烧录 boot.img。如果用户需要将开机图片替换为自己的 logo，首先需要将你的 U-Boot logo 图片重命名为 logo.bmp、
将你的内核 logo 图片重命名为 logo_kernel.bmp，然后将这两个 bmp 图片文件拷贝到内核源码根目录下，替换内核源码中默认的 logo 图片。编译内核时，会将logo.bmp和logo_kernel.bmp打包进resource.img中，然后再把resource.img打包到 boot.img。U-Boot 启动的时候会把这两个 bmp 文件加载到内存中，logo.bmp 会在 U-Boot 阶段开始显示，logo_kernel.bmp 在内存中的地址会被 U-Boot 传给 Linux Kernel，在 Linux Kernel 的 DRM驱动初始化阶段显示。

# 五、buildroot 开发

Buildroot 源码在<SDK>/buildroot 目录下：
![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_7.png)

| 顶层文件名    | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| **arch**      | 存放 buildroot 支持的所有 CPU 架构相关的配置文件及构建脚本   |
| **board**     | 存放特定目标平台相关的文件，譬如内核配置或补丁文件、rootfs覆盖文件等 |
| **boot**      | 存放 buildroot 支持的 BootLoader 相关的补丁、校验文件、构建脚本、配置选项等 |
| **build**     | Buildroot 编译系统相关组件                                   |
| **configs**   | 存放了所有目标平台的 defconfig 配置文件                      |
| **dl**        | download 的缩写，该目录用于存放下载的各种开源软件包，譬如alsa-lib 库、bluez 库、bzip2、curl 工具等；在编译过程中，buildroot会从网络下载所需软件包、并将其放置在 dl/目录下；如果 buildroot下载某软件包时失败、无法下载成功，此时我们也可以自己手动下载该软件包、并将其拷贝至 dl 目录下。所有软件包只需下载一次即可，不是每次编译都要下载一次（除非删除 dl 目录），只要 dl 目录下存在该软件包就不用下载了；所以往往第一次编译会比较慢，因为下载过程会占用很多时间 |
| **docs**      | 存放相关的参考文档                                           |
| **fs**        | 存放各种文件系统的源代码                                     |
| **linux**     | 存放 linux 的构建脚本和配置选项                              |
| **output**    | 该文件夹会在编译 buildroot 后出现，output 目录用于存放编译过程中输出的各种文件，包括各种编译生成的中间目标文件、可执行文件、lib 库以及最终烧录到开发板的 rootfs 镜像等 |
| **package**   | 存放所有 package（软件包）的构建脚本、配置选项；每个软件包目录下（package/<package_name>/）都有一个 Config.in 文件和<package_name>.mk 文件（其实就是 Makefile 文件）；如果需要添加一个新的 package，则需对 package/目录进行改动。 |
| **support**   | 存放一些为 buildroot 提供功能支持的脚本、配置文件等          |
| **toolchain** | 存放制作各种交叉编译工具链的构建脚本和相关文件，binutils、gcc、gdb、kernel-header 和 uClibc |
| **utils**     | 存放一些 buildroot 的实用脚本和工具                          |

## 5.1 常见编译命令

在SDK里面编译buildroot 源码有两种常用的方法，一种是在SDK顶层目录直接用命令`./build.sh buildroot`,还一种是依赖buildroot 源码根目录下的 Makefile 文件。

编译之前，先进行配置；进入到 buildroot 目录下，执行如下命令进行配置

```
lhd@ydtx:~/project_code/3588/3588_linux5.10_v1.0.5/buildroot$ source build/envsetup.sh rockchip_rk3588
Top of tree: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5
===========================================

#TARGET_BOARD=rk3588
#OUTPUT_DIR=output/rockchip_rk3588
#CONFIG=rockchip_rk3588_defconfig

===========================================
make: 进入目录“/home/lhd/project_code/3588/3588_linux5.10_v1.0.5/buildroot”
  GEN     /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/buildroot/output/rockchip_rk3588/Makefile
/home/lhd/project_code/3588/3588_linux5.10_v1.0.5/buildroot/build/parse_defconfig.sh /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/buildroot/configs/rockchip_rk3588_defconfig /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/buildroot/output/rockchip_rk3588/.config.in
Parsing defconfig: /home/lhd/project_code/3588/3588_linux5.10_v1.0.5/buildroot/configs/rockchip_rk3588_defconfig
Using configs/rockchip/chips/rk3588.config as base
Merging configs/rockchip/chips/rk3588_aarch64.config
Merging configs/rockchip/base/kernel.config
Merging configs/rockchip/e2fs.config
Merging configs/rockchip/base/common.config
Merging configs/rockchip/base/base.config
Value of BR2_ROOTFS_OVERLAY is redefined by configs/rockchip/base/base.config:
Previous value:	BR2_ROOTFS_OVERLAY="board/rockchip/rk3588/fs-overlay/"
Modify value:	BR2_ROOTFS_OVERLAY+="board/rockchip/common/base"
New value:	BR2_ROOTFS_OVERLAY="board/rockchip/rk3588/fs-overlay/ board/rockchip/common/base"
```

命令中最后一个参数（rockchip_rk3568）用于指定目标平台的 defconfig 配置文件（不带_defconfig 后缀），所有目标平台的 defconfig 配置文件都存放在<Buildroot>/configs 目录下

配置完成后，直接执行 `make` 或`make all`命令编译根文件系统,如下所示为部分内容：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_8.png)

编译 过程中，会编译两次 buildroot：

- 编译 buildroot 得到根文件系统镜像 rootfs.img。rootfs.img 会烧录到开发板 rootfs 分
  区。
- 编译 buildroot 得到 ramdisk 根文件系统镜像（进入 recovery 模式时挂载 ramdisk 根文
  件系统，ramdisk 最终会打包进 recovery.img 镜像中，recovery.img 会烧录到开发板 recovery 分
  区。）

Buildroot内置交叉编译，其交叉编译⼯具位于buildroot/output/rockchip_rk3588/host/usr ⽬录下，输⼊命令查看：

```
cd buildroot/output/rockchip_rk3588/host/usr/bin
./aarch64-linux-gcc --version
aarch64-linux-gcc.br_real (Buildroot -g167e3f26b-dirty) 11.3.0
```

**若需要编译单个模块或者第三⽅应⽤⽐如 rockchip-test 模块，常⽤相关编译命令如下：**

- 编译rknpu2
  SDK$make rknpu2
- 重编rknpu2
  SDK$make rknpu2-rebuild
- 删除rknpu2
  SDK$make rknpu2-dirclean
  或者
  SDK$rm -rf /buildroot/output/rockchip_rk3588/build/rknpu2-1.0.0

## 5.2 output 目录介绍

output 目录用于存放编译过程中产生的各种文件，包括编译过程生成的中间目标文件、可
执行文件、lib 库以及最终烧录到开发板的 rootfs 镜像等。

> output/
> ├── rockchip_rk3588
> │ ├── build #包含所有构建的软件包，包括 buildroot 在宿主机上所需的工具以
> 及为目标平台编译的软件包
> │ ├── host #包含为 Ubuntu 主机（宿主机）构建的工具，以及目标工具链
> │ ├── images #存放最终编译输出的镜像
> │ ├── Makefile
> │ ├── staging #一个指向 host/目录中目标工具链 sysroot 的符号链接
> │ └── target #根文件系统系统目录，用来创建根文件系统镜像，rootfs.img 镜像
> 的内容就是该目录下的内容

## 5.3 软件包编译

如何编译指定的软件包（package），在 buildroot/package/文件夹中的某个目录对应的名字（或某个子目录对应的名字），并且该目录中存在 Config.in 配置文件（用于定义 package 配置选项）以及<package_name>.mk 文件（Makefile 文件，用于定义 package 的构建逻辑）。譬如：

| rkwifibt → package/rockchip/rkwifibt/<br/>rknpu → package/rockchip/rknpu/<br/>rkupdate → package/rockchip/rkupdate/ |
| ------------------------------------------------------------ |

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_9.png)

接着执行命令编译指定的 package：`make <package_name>`

package（软件包）的构建过程可以分解为：configure（配置）、build（编译）、install（安装），
其执行顺序为：配置→编译→安装，每一步可单独执行，执行如下命令：

```
make <package_name>-configure #执行配置命令
make <package_name>-build #执行编译命令
make <package_name>-install #执行安装命令
```

## 5.4 添加 package软件包

下面通过一个简单的例子向用户介绍如何在 buildroot/package 目录下添加一个自己的package（软件包）。

### 5.4.1 开发源码工程

首先进入<SDK>/app 目录下，在该目录下创建一个名为“packagedemo”的文件夹,且在 packagedemo 目录下创建一个.c 源文件 main.c，以及一个 Makefile 文件：

```
cd <SDK>/app
mkdir packagedemo
touch main.c Makefile
```

在 main.c 源文件中编写一个简单的测试代码，譬如打印一个“Hello World”。
Makefile 文件中的内容如下所示：

```
package: main.o
$(CC) -o package main.o
%.o: %.c
$(CC) -c $< -o $@
```

目的就是将 main.c 源文件编译成一个可执行文件 package。

### 5.4.2 添加 package

进入<Buildroot>/package 目录，在该目录下创建一个名为 packagedemo 的目录,并且在packagedemo目录下创建两个文件：Config.in 和 packagedemo.mk。

Config.in 文件的内容如下所示：

```
config BR2_PACKAGE_PACKAGEDEMO
bool "PACKAGEDEMO"
help
 this configuration is used to enable or disable packagedemo.
```

packagedemo.mk 文件的内容如下所示：

```
################################################################################
#
# packagedemo
#
################################################################################
# 给你的软件包定义一个版本号
MYPACKAGE_VERSION = 1.0
# 你的软件包所在目录
MYPACKAGE_SITE = $(TOPDIR)/../app/packagedemo
# 获取软件包的方式，local 表示从本地获取，有些包可能需要通过网络下载，譬如 git 仓库中
的项目
MYPACKAGE_SITE_METHOD = local
# 列出在编译软件包之前 需要执行的配置操作
define MYPACKAGE_CONFIGURE_CMDS
endef
# 列出编译软件包时 需要执行的操作
define MYPACKAGE_BUILD_CMDS
$(MAKE) -C $(@D) CC=$(TARGET_CC)
endef
# 列出将软件包安装到 target 目录(<Buildroot>/output/rockchip_rk3588/target)时需要执行的操作
define MYPACKAGE_INSTALL_TARGET_CMDS
$(INSTALL) -D -m 0755 $(@D)/mypackage $(TARGET_DIR)/usr/bin/package
endef
# 表示当前软件包是一个通用型软件包基础结构
$(eval $(generic-package))
```

注意：该文件中定义了一些变量以及宏，所有的这些变量、宏都以前缀 MYPACKAGE_开头，不能乱来，它必须等于 Config.in、mypackage.mk 文件所在目录（mypackage）对应的名字（小写字母转换为大写）。
$(MAKE)：表示 make 命令；
$(@D)：表示软件包所在目录，注意这个目录并不是<SDK>/app/mypackage、而是该软件包在 output/rockchip_rk3588/build/目录下对应的文件夹；编译软件包之前，buildroot 会将<SDK>/app/mypackage 拷 贝 至 <Buildroot>/output/rockchip_rk3588/build/ 目 录 ， 并 重 命 名 为mypackage-1.0（1.0 就是版本号）。所以这个“$(@D)”指的是 output/rockchip_rk3588/build/mypackage-1.0 这个目录。
$(TOPDIR)：表示 buildroot 顶层目录，也就是<SDK>/buildroot 目录。
$(TARGET_CC)：表示交叉编译器，RK 平台默认使用 buildroot 交叉编译器，交叉编译器所在路径为：<Buildroot>/output/rockchip_rk3588/host/bin/aarch64-buildroot-linux-gnu-gcc。
$(INSTALL)：表示 install 命令。
$(TARGET_DIR)：表示 target 目录<Buildroot>/output/rockchip_rk3588/target。



接下来打开 package/Config.in 文件，将下面这行内容添加到该文件中

```
source "package/packagedemo/Config.in"
```

添加完成后保存退出。



## 5.4.3 使能并编译 package

执行“make menuconfig”打开图形化配置界面，输入“/”搜索“packagedemo”找到我们添加的 package，将其使能并且保存配置、退出图形化配置界面。

执行`make packagedemo-rebuild`命令编译该软件包

编 译会 生 成 一 个 可 执 行 文 件 package ， 其 所 在 路 径 为 ：output/rockchip_rk3588/target/usr/bin/package



# 六、Debian开发

源码位于⼯程 <SDK>/debian ⽬录下

```
debian
├── mk-base-debian.sh ##获取Debian基础包和编译
├── mk-image.sh ##打包⽣成ext4的固件
├── mk-rootfs-buster/bullseye.sh ##适配Rockchip相关硬件加速包
├── mk-rootfs.sh ##指向具体Rootfs版本，⽬前有Buster、Bullseye两个版本。
├── overlay ##适配Rockchip平台共性配置⽂件
├── overlay-debug ##系统常使⽤的调试⼯具
├── overlay-firmware ##⼀些设备firmware的存放，⽐如npu/dp等
├── packages ## 包含armhf arm64系统适配硬加速使⽤的预编译的包
├── packages-patches ##预编包，基于官⽅打上的补丁
├── readme.md ## ⽂档指引
└── ubuntu-build-service ##从官⽅获取Debian发⾏版，可依赖包和定制安装相关包。
```

整个⽬录结构内容是通过Shell脚本来达到获取构建Linux Debian发⾏版源码

## 6.1 Debian 编译

Debian常用的的编译方式有两种，一种是在SDK目录下执行`./build.sh debian`,或者进入 debian/ ⽬录执行编译命令。

编译和 Debian 固件⽣成参考当前⽬录 readme.md

````
## Introduction

A set of shell scripts that will build GNU/Linux distribution rootfs image
for rockchip platform.

## Available Distro

* Debian 11 (Bullseye-X11 and Wayland)~~

```
sudo apt-get install binfmt-support qemu-user-static
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f
```

## Usage for 32bit Debian 11 (Bullseye-32)

### Building debian system from linaro

Building a base debian system by ubuntu-build-service from linaro.

```
	RELEASE=bullseye TARGET=base ARCH=armhf ./mk-base-debian.sh
```

Building a desktop debian system by ubuntu-build-service from linaro.

```
	RELEASE=bullseye TARGET=desktop ARCH=armhf ./mk-base-debian.sh
```

### Building overlay with rockchip audio/video hardware accelerated

Building with overlay with rockchip debian rootfs:

```
	RELEASE=bullseye ARCH=armhf ./mk-rootfs.sh
```

Building with overlay with rockchip debug debian rootfs:

```
	VERSION=debug ARCH=armhf ./mk-rootfs-bullseye.sh
```

### Creating roofs image

Creating the ext4 image(linaro-rootfs.img):

```
	./mk-image.sh
```

---

## Usage for 64bit Debian 11 (Bullseye-64)

Building a base debian system by ubuntu-build-service from linaro.

```
	RELEASE=bullseye TARGET=desktop ARCH=arm64 ./mk-base-debian.sh
```

Building the rk-debian rootfs:

```
	RELEASE=bullseye ARCH=arm64 ./mk-rootfs.sh
```

Building the rk-debain rootfs with debug:

```
	VERSION=debug ARCH=arm64 ./mk-rootfs-bullseye.sh
```

Creating the ext4 image(linaro-rootfs.img):

```
	./mk-image.sh
```
---

## Cross Compile for ARM Debian

[Docker + Multiarch](http://opensource.rock-chips.com/wiki_Cross_Compile#Docker)

## Package Code Base

Please apply [those patches](https://github.com/rockchip-linux/rk-rootfs-build/tree/master/packages-patches) to release code base before rebuilding!

## License information

Please see [debian license](https://www.debian.org/legal/licenses/)

## FAQ

- noexec or nodev issue
noexec or nodev issue /usr/share/debootstrap/functions: line 1450:
../rootfs/ubuntu-build-service/bullseye-desktop-arm64/chroot/test-dev-null:
Permission denied E: Cannot install into target
...
mounted with noexec or nodev

Solution: mount -o remount,exec,dev xxx (xxx is the mount place), then rebuild it.
````

当执行完./mk-image.sh之后，会在当前目录下生成Debian固件linaro-rootfs.img。

## 6.2 系统定制

Debian系统是通过Shell脚构建出来的，要对Debian系统进行定制，需要在<SDK>/debian ⽬录下着重关注两个脚本：mk-base-debian.sh和mk-rootfs-bullseye.sh

mk-base-debian.sh脚本部分内容如下：

```
cd ubuntu-build-service/$RELEASE-$TARGET-$ARCH

echo -e "\033[36m Staring Download...... \033[0m"

make clean

./configure

make

if [ -e linaro-$RELEASE-alip-*.tar.gz ]; then
	sudo chmod 0666 linaro-$RELEASE-alip-*.tar.gz
	mv linaro-$RELEASE-alip-*.tar.gz ../../
else
	echo -e "\e[31m Failed to run livebuild, please check your network connection. \e[0m"
fi
```

构建了Debian的linaro-bullseye-alip-20230613-1.tar.gz基础包



![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588基本使用文档\03_SDK开发\SDK_10.png)

对系统的软件包进⾏定制：将所需的包列表放在customization/package-lists⽬录下，并命名为XXX.list.chroot或XXX.list.binary，如果该目录下已经有这个文件，可以直接对其进行修改。

mk-rootfs-bullseye.sh适配相关硬件加速包，也可以在这里添加自己的应用，部分脚本如下：

```
#---------Camera---------
echo -e "\033[36m Install camera.................... \033[0m"
\${APT_INSTALL} cheese v4l-utils
\${APT_INSTALL} /packages/libv4l/*.deb

#---------armsom-w3-test---------
# echo -e "\033[36m Install armsom-w3-test.................... \033[0m"
# \${APT_INSTALL} /packages/armsom-w3-test/*.deb

#---------Xserver---------
echo -e "\033[36m Install Xserver.................... \033[0m"
\${APT_INSTALL} /packages/xserver/*.deb
```

armsom-w3-test是armsom自写的一个产测软件，在debian/packages/arm64下创建armsom-w3-test文件夹，将test.deb放在文件夹下

再执行相关脚本就可以将我们的应用打包进系统，mk-rootfs-bullseye.sh这里有很多预安装包，这里就不解释了，已经预安装的Debian deb包可以对其进行解压、修改、重新打包，这里提供一种⽅法：

```
#解压出包中的⽂件到extract⽬录下
dpkg -X xxx.deb extract/
#解压出包的控制信息extract/DEBIAN/下：
dpkg -e xxx.deb extract/DEBIAN/
#修改⽂件XXX
# 对修改后的内容重新进⾏打包⽣成deb包
dpkg-deb -b extract/ .
```





