# 第一章 构建 Debian Linux 系统

我们需要按【01_RK3588编译&烧录Linux固件】全自动编译一次，默认是编译 Buildroot 系统，也会编
译 uboot 和内核，buildroot 某些软件包依赖内核，所以我们必须编译内核再编译 Buildroot。同
理 Debian 也需要从 Buildroot 编译后的产物，拷贝相关软件到 Debian 中，一般是一些驱动模块。
所以它们的编译关系是不可分开的。

## 1.1 Debian 安装 qemu

构建Debian系统需要依赖你电脑本地的环境，有些依赖包需要下载

在 SDK 顶层目录，进入 debian 路径下

```
cd debian/
```

![image-20230831144532865](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230831144532865.png)

执行下面的指令安装 qemu 及需要的一些包

```
sudo apt-get install binfmt-support qemu-user-static live-build
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f
```

![image-20230831145336413](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230831145336413.png)

## 1.2 构建 Debian 系统

执行下面指令开始构建 Debian 系统。注意，如果你没有先编译 buildroot 和内核，虽然不会
报错，完整的系统会从 buildroot 和内核中拷贝一些驱动或者文件到 Debian 系统中，拷贝的文
件是 Wifi 驱动和蓝牙驱动。（**注意构建的时候 Ubuntu 必须能上网，因为 Debian 会从网上下载**
**构建所需要的资源**！）

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

- mk-base-debian.sh
  在Debian官网下载基础包，有些系统组件不需要的话可以在这里修改定制

- mk-rootfs-bullseye.sh

  二次定制Debian系统，主要是一些已经打包好了的安装包，在这里你也可以加载你需要的应用软件

- mk-image.sh

  打包Debian系统为img文件



构建过程中，首先会下载 Debian 系统，下载完 Debian 系统后，开始解压 Debian 系统，
此时需要用户权限，请输入你的用户密码，提升为 sudo 权限再继续编译。（**注意千万不要为了**
**省事直接以 root 用户直接构建！避免出现离奇的错误！**）

## 1.3 如何使用编译出来的 linaro-rootfs.img

上述编译在debian目录下生成linaro-rootfs.img，

### 1.3.1 分区烧录

可以参考[分区烧录](https://youtu.be/UExqUcTjCkI)视频，在烧录工具中单独勾选“rootfs”位置

### 1.3.2 完整Debian系统

上面编译出来的只是文件系统，无法生成 update.img，如果你要生成 update.img，在SDK首目录执行下
面的操作。

```
export RK_ROOTFS_SYSTEM=debian 	# 构建系统类型为 debian
./build.sh 						# 执行./build.sh 会生成 update.img
```

构建的过程中会编译 Uboot、内核等，等等漫长的时间后，最后会将 Uboot、内核和文件系
统等打包生成 update.img。
生成的 update.img 在 SDK/rockdev 下。

# 第二章 Debian 系统开发

Debian 是一种完全自由开放并广泛用于各种设备的 Linux 操作系统。选择 Debian 原因如
下，Debian 是自由软件并且保持 100%自由。每个人都能自由使用、修改，以及发布。大家可
以基于 Rockchip 构建的 Debian 系统进行二次开发。
Debian 是一个基于 Linux 稳定且安全的操作系统，其使用范围包括笔记本计算机台式机和
服务器等。它的稳定性和可靠性就深受用户的喜爱。
Debian 还是许多其他发行版的种子和基础，例如 Ubuntu、Knoppix、PureOS 等，由世界各
种的数百名志愿者共同制作。
目前 Rockchip RK3588 已经适配并支持。

## 2.1 Debian 版本

输入下面的指令查看 Debian 版本，如下可以看到是 Debian11 版本。

```
cat /etc/issue
```

![image-20230831173008985](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230831173008985.png)

输入下面的指令查看 内核 版本

```
uname -a
```

![image-20230831173056799](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230831173056799.png)





## 2.2 如何更新源

所谓的源，就是源头，我们常见的源有阿里云源（mirrors.aliyun.com），中科大源
（mirrors.ustc.edu.cn），南京大学（mirror.nju.edu.cn）源，清华大学源（mirrors.tuna.tsinghua.edu.cn），
还有 Debian 官方源（deb.debian.org）等都提供了对 Debian 源的服务。在 Rockchip 中默认使用
的是中科大源（mirrors.ustc.edu.cn）。实际上 Rockchip 已经为我们由 Debian 官方源切换为中科
大源，只要是国内的源，下载速度都是较快的。我们可以无需再换源。如果我们还是想试试换
其他源，可以按以下的方法来更换其它源。
查看默认的源

```
cat /etc/apt/sources.list
```

![image-20230831173957487](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230831173957487.png)

修改为其他源，例如阿里云源，如下，替换完成保存。如需要替换成其它源，可以自行尝试。
编辑/etc/apt/sources.list 文件，替换成如下内容。

```
deb http://mirrors.aliyun.com/debian buster main contrib non-free
deb-src http://mirrors.aliyun.com/debian buster main contrib non-free
deb http://mirrors.aliyun.com/debian-security buster/updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian-security buster/updates main contrib non-free
deb http://mirrors.aliyun.com/debian buster-updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian buster-updates main contrib non-free
```

执行下面指令更新本地仓库，也就是重定向软件包列表。更新软件源，记得插网线！并且网线可以上网的！

```
apt-get upgrade
apt-get update
```

## 2.3 软件包管理

查看系统已经安装的软件

```
apt list --installed
```

查看可安装的软件，执行下面的指令查看可安装的软件，若软件已经安装会显示已安装。

```
apt list
```

使用 apt remove 来卸载软件。apt 会解决和安装模块的依赖问题,并会咨询软件仓库, 但不会

安装本地的 deb 文件, apt 是建立在 dpkg 之上的软件管理工具。所以我们一般用 apt 管理软件。

例如我们卸载 vim 软件。首先我们查看 vim 是否已经安装。可以执行下面指令。

```
apt list --installed | grep vim
```

![image-20230831175054894](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230831175054894.png)

上图可以看到 vim 已经安装，那么我们将它卸载。询问是否卸载时，我们输入“y”确认卸载

同样我们使用 apt 来安装软件，执行下面的指令安装 vim。

```
apt install vim
```

## 2.4 如何创建自启动程序

在 debian 系统很多时候都有开机自启动自己的程序，这里有两种方法

### 2.4.1 /etc/rc.local 方法

通过将自启动程序放到/etc/rc.local 里，开机就会自启动，也可以写到/etc/init.d 下面
的某个文件里。

这里以/etc/rc.local 文件为例。如下图，打印“1111111111111111111111”，以此观察系统启动时，是
否会自动执行。注意我们的程序不能写到 exit0 后面了！

![image-20230901094324408](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230901094324408.png)

保存后重启开发板，可以看到启动后系统执行了这句打印！

![image-20230901094507296](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230901094507296.png)

### 2.4.2 添加systemd方法

我们先创建一个要启动的脚本。这个脚本就是打印“22222……”在/root 目录下编辑auto_run_script.sh。

```
#!/bin/bash
echo “2222222222222222222222222222” > /dev/ttyFIQ0
```

赋予脚本可执行权限，执行下面的指令

```
chmod +x auto_run_script.sh
```

- /lib/systemd/system 是系统范围的目录，这些文件通常是由发行版的维护者创建的。
- /etc/systemd/system：这个目录也包含 systemd 服务单元文件，但是它是用于本地管理员（系
  统管理员）自定义的服务配置。不会被/lib/systemd/system 服务覆盖。

所以我们需要进入/etc/systemd/system 目录下，创建一个自启动服务 auto_run_script.service。

```
cd /etc/systemd/system
vi auto_run_script.service
```

在 auto_run_script.service 里添加以下内容。

```
[Unit]
Description=Run a Custom Script at Startup
After=network.target
[Service]
ExecStart=/root/auto_run_script.sh
[Install]
WantedBy=default.target
```

然后我们使用 systemctl daemon-reload 重新加载 systemd 守护进程配置，并使用 systemctl 
enable 建立符号链接关系。

```
systemctl daemon-reload
systemctl enable auto_run_script.service
systemctl start auto_run_script.service #马上启动此服务
```

重启后，我们看看打印信息，查看这个服务是否已经启动

如需禁止自启动，输入 systemctl disable auto_run_script.service，这个就会取消链接，下
次开机时不会自启动。如需要完全移除，删除这个 auto_run_script.service 以及它的脚本。