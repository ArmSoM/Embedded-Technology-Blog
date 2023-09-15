# 1. 简介
- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)
- Debian 是⼀种完全⾃由开放并⼴泛⽤于各种设备的 Linux 操作系统。
- Rockchip在官⽅Debian发⾏版的基础上构建和适配了相关硬件功能

# 2. 环境介绍

- **硬件环境：**
ArmSoM-W3 RK3588开发板

- **软件版本：**
OS：ArmSoM-W3 Debian11
# 3. Debian目录结构
debian
├── mk-base-debian.sh ##获取Debian基础包和编译
├── mk-rootfs-buster/bullseye.sh ##在Debian基础包的基础上适配Rockchip相关硬件加速包
├── mk-image.sh ##⽣成ext4的固件(生成linaro-rootfs.img)
├── mk-rootfs.sh ##指向具体Rootfs版本，⽬前有Buster、Bullseye两个版本。
├── overlay ##适配Rockchip平台共性配置⽂件。overlay目录会覆盖到根文件系统，来满足客制化的需求
├── overlay-debug ##系统常使⽤的调试⼯具
├── overlay-firmware ##⼀些设备firmware的存放，⽐如npu/dp等
├── packages ## 包含armhf arm64系统适配硬加速使⽤的预编译的包
├── packages-patches ##预编包，基于官⽅打上的补丁
├── scripts ## 编译，安装，打包的脚本
├── readme.md ## ⽂档指引
└── ubuntu-build-service ##从官⽅获取Debian发⾏版，可依赖包和定制安装相关包。

**整个⽬录结构内容是通过Shell脚本来达到获取构建Linux Debian发⾏版源码，编译和安装适配Rockchip硬加速包的操作系统。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/e0cfe08b3ac0493abc6abcd8d9c8e455.png)




# 4. Debian编译和烧录
## 4.1 Debian编译方式1：
- 最简单的方法就是SDK根目录下直接运行编译脚本

	```bash
	./build.sh debian
	```
	
- 编译成功后会在rockdev目录下生成根文件镜像rootfs.img，用RKDevTool烧录即可。
## 4.2 Debian编译方式2：
- 进⼊ debian/ ⽬录：

	```bash
	cd debian/
	```
- 第1步：构建64 位的基础 Debian 系统

	```bash
	RELEASE=bullseyeTARGET=desktop ARCH=arm64 ./mk-base-debian.sh
	```
	编译完成会在 debian/ ⽬录下⽣成：linaro-bullseye-alip-xxxxx-1.tar.gz（xxxxx 表⽰⽣成时间戳)。

- 第2步：构建 rk-debian rootfs （增加Rockchip相关配置适配包括相关硬件加速包）
	
	```bash
	VERSION=debug ARCH=arm64 ./mk-rootfs-bullseye.sh
	```
- 第3步：创建 ext4 镜像(linaro-rootfs.img)，将编译后生成的binary根文件打包⽣成ext4的固件(生成linaro-rootfs.img)
	```bash
	./mk-image.sh
	```
## 4.3 build_debian函数
- build.sh中的build_debian函数如下，可以看出是根据条件执行了mk-base-debian.sh和mk-rootfs-$RK_DEBIAN_VERSION.sh两个脚本，前者是Debian基础包和编译，后者是增加Rockchip相关配置适配。

	```cpp
	build_debian()
	{
		ARCH=${RK_DEBIAN_ARCH:-${RK_KERNEL_ARCH}}
		case $ARCH in
			arm|armhf) ARCH=armhf ;;
			*) ARCH=arm64 ;;
		esac
	
		echo "=========Start building debian ($ARCH) rootfs========="
	
		cd debian
		if [ ! -f linaro-$RK_DEBIAN_VERSION-alip-*.tar.gz ]; then
			RELEASE=$RK_DEBIAN_VERSION TARGET=desktop ARCH=$ARCH ./mk-base-debian.sh
			ln -rsf linaro-$RK_DEBIAN_VERSION-alip-*.tar.gz linaro-$RK_DEBIAN_VERSION-$ARCH.tar.gz
		fi
	
		VERSION=debug ARCH=$ARCH ./mk-rootfs-$RK_DEBIAN_VERSION.sh
		./mk-image.sh
	
		finish_build
	}
	```

# 5. 系统基本信息查看
## 5.1 系统版本

```cpp
root@linaro-alip:~# cat /etc/debian_version 
11.6
```
## 5.2 如何查看Debian显⽰⽤X11还是Wayland？
在X11系统上：

```bash
$ echo $XDG_SESSION_TYPE
x11
```
Wayland系统上：
```bash
$ echo $XDG_SESSION_TYPE
wayland
```
## 5.3 如何查看系统分区情况

```bash
parted -l
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0ab284b4c01f41848acb3c5c85827e4a.png)

## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)