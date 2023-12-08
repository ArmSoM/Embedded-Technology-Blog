# armbian开发指北（一）
## 1. 什么是armbian

Armbian是一个基于Debian或Ubuntu的开源操作系统，专门针对嵌入式ARM平台进行优化和定制。Armbian可以运行在多种不同的嵌入式设备上，例如树莓派、[ArmSoM](https://www.armsom.org/)、香蕉派等等。Armbian针对不同的嵌入式平台，提供了相应的硬件支持，可以让用户轻松地在这些平台上搭建自己的嵌入式系统。

[armbian](https://www.armbian.com/)立项于2014年底，于2016年开始进入频繁更新，每年千万行代码的爆发式成长，截止目前为止，官网已经支持185个不同的硬件设备的适配。

![armbian-office-web](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armbianarmbian-office-web.png)

## 2. 为什么要使用armbian

Armbian提供了丰富的软件库和组件，包括Linux内核、文件系统、应用程序等，用户可以根据自己的需要进行选择和安装。Armbian还提供了一套完整的开发工具链，方便用户进行开发和调试工作。

总的来说，Armbian是一款功能强大、灵活性高、易于定制的嵌入式操作系统，适用于各种不同的嵌入式设备和应用场景。

## 3. 如何使用armbian
### 3.1 基本要求
* x86_64 或 aarch64 计算机，至少具有 2GB 内存和 ~35GB 磁盘空间，用于虚拟机、WSL2、容器或裸机安装
* Ubuntu Jammy 22.04.x amd64 或 aarch64 用于本机构建或任何支持 Docker 的 amd64 / aarch64 Linux 用于容器化。官方支持的编译环境仅限 Ubuntu Jammy 22.04.x amd64 ！
* 超级用户权限（配置的 sudo 或 root 访问权限）。
* 确保您的所有系统组件都是最新的。例如，过时的 Docker 二进制文件可能会导致问题。
* 很多资源包下载都需要外网，中国的用户需要一个翻墙工具

### 3.2 开始构建
```
$ apt-get -y install git
$ git clone --depth=1 --branch=main https://github.com/armbian/build
$ cd build
$ ./compile.sh
```

命令执行后会进行以下三个操作，具体的操作解释，后续我会写文章详细解释
* 交互式图形界面。
* 通过安装必要的依赖项和源来准备工作区。
* 它指导整个过程并创建内核包或即用型 SD 卡映像。

### 4. 项目结构
```
├── cache                                Work / cache directory
│   ├── aptcache                         Packages
│   ├── ccache                           C/C++ compiler
│   ├── docker                           Docker last pull
│   ├── git-bare                         Minimal Git
│   ├── git-bundles                      Full Git
│   ├── initrd                           Ram disk
│   ├── memoize                          Git status
│   ├── patch                            Kernel drivers patch
│   ├── pip                              Python
│   ├── rootfs                           Compressed userspaces
│   ├── sources                          Kernel, u-boot and other sources
│   ├── tools                            Additional tools like ORAS
│   └── utility
├── config                               Packages repository configurations
│   ├── targets.conf                     Board build target configuration
│   ├── boards                           Board configurations
│   ├── bootenv                          Initial boot loaders environments per family
│   ├── bootscripts                      Initial Boot loaders scripts per family
│   ├── cli                              CLI packages configurations per distribution
│   ├── desktop                          Desktop packages configurations per distribution
│   ├── distributions                    Distributions settings
│   ├── kernel                           Kernel build configurations per family
│   ├── sources                          Kernel and u-boot sources locations and scripts
│   ├── templates                        User configuration templates which populate userpatches
│   └── torrents                         External compiler and rootfs cache torrents
├── extensions                           Extend build system with specific functionality
├── lib                                  Main build framework libraries
│   ├── functions
│   │   ├── artifacts
│   │   ├── bsp
│   │   ├── cli
│   │   ├── compilation
│   │   ├── configuration
│   │   ├── general
│   │   ├── host
│   │   ├── image
│   │   ├── logging
│   │   ├── main
│   │   └── rootfs
│   └── tools
├── output                               Build artifact
│   └── deb                              Deb packages
│   └── images                           Bootable images - RAW or compressed
│   └── debug                            Patch and build logs
│   └── config                           Kernel configuration export location
│   └── patch                            Created patches location
├── packages                             Support scripts, binary blobs, packages
│   ├── blobs                            Wallpapers, various configs, closed source bootloaders
│   ├── bsp-cli                          Automatically added to armbian-bsp-cli package
│   ├── bsp-desktop                      Automatically added to armbian-bsp-desktopo package
│   ├── bsp                              Scripts and configs overlay for rootfs
│   └── extras-buildpkgs                 Optional compilation and packaging engine
├── patch                                Collection of patches
│   ├── atf                              ARM trusted firmware
│   ├── kernel                           Linux kernel patches
|   |   └── family-branch                Per kernel family and branch
│   ├── misc                             Linux kernel packaging patches
│   └── u-boot                           Universal boot loader patches
|       ├── u-boot-board                 For specific board
|       └── u-boot-family                For entire kernel family
├── tools                                Tools for dealing with kernel patches and configs
└── userpatches                          User: configuration patching area
    ├── lib.config                       User: framework common config/override file
    ├── config-default.conf              User: default user config file
    ├── customize-image.sh               User: script will execute just before closing the image
    ├── atf                              User: ARM trusted firmware
    ├── kernel                           User: Linux kernel per kernel family
    ├── misc                             User: various
    └── u-boot                           User: universal boot loader patches
 CONTRIBUTING.md // We would love to have you join the Armbian 
```

## 4. 感谢

此系列特别感谢 [armsom团队](https://github.com/armsom) 和 armbian中国开发者 [amazingfate](https://github.com/amazingfate) 
