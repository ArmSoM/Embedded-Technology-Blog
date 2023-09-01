# RK3588编译&烧录Linux固件

## 1、开发环境及工具准备

Rockchip Linux 软件包：linux-5.10-gen-rkr4

主机：

- 安装VMware搭建虚拟机，版本为Ubuntu 20.04 (硬盘容量大于100G）

- 安装远程连接工具MobaXterm（可连接虚拟机方便文件传输）


## 2、SDK编译环境搭建

### 1、安装库和工具集：

```shell
sudo apt-get install git ssh make gcc libssl-dev liblz4-tool expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib unzip device-tree-compiler ncurses-dev libgucharmap-2-90-dev bzip2 expat gpgv2 cpp-aarch64-linux-gnu time mtd-utils
```


### 2、创建工作目录

```
mkdir ~/RK3588
```


### 3、拷贝SDK至工作目录

可通过MobaXterm实现PC与虚拟机之间传输文件


### 4、解压SDK

解压命令:
```
cat linux-5.10-gen-rkr4.tar.gzaa* | tar xzvf -
```


### 5、检查和升级软件包

- 检查make版本(要求make 4.0及以上版本）
```
make -v
GNU Make 4.2
Built for x86_64-pc-linux-gnu
```

- 升级make版本
```
git clone https://github.com/mirror/make.git
cd make
git checkout 4.2
git am $BUILDROOT_DIR/package/make/*.patch
autoreconf -f -i
./configure
make make -j8
sudo install -m 0755 make /usr/bin/make
```


- 检查lz4版本（要求安装 lz4 1.7.3及以上版本）
```
lz4 -v
*** LZ4 command line interface 64-bits v1.9.4, by Yann Collet ***
refusing to read from a console
```

- 升级lz4版本
```
git clone https://github.com/lz4/lz4.git
cd lz4
make
sudo make install
sudo install -m 0755 lz4 /usr/bin/lz4
```


- 检查和升级git版本
```
git clone https://github.com/mirror/make.git --depth 1 -b 4.2
cd make
git am $BUILDROOT_DIR/package/make/*.patch
autoreconf -f -i
./configure
make make -j8
install -m 0755 make /usr/local/bin/make
```


### 6、git配置

在~/RK3588/linux-5.10-gen-rkr4目录下
```
git config --global user.name "your name"
git config --global user.email "your mail"
```


### 7、安装repo

```
mkdir ~/bin
export PATH=~/bin:$PATH
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo
chmod a+x ~/bin/repo
```


## 3、编译SDK

可参考 device/rockchip/common/README.md 编译说明。

### SDK编译命令查看
```
make help
```

### SDK配置：
可通过``` make lunch ```或者``` ./build.sh lunch ```进⾏配置，其他功能的配置可通过``` make menuconfig ```来配置相关属性


### 全自动编译

进⼊~/RK3588/linux-5.10-gen-rkr4目录执⾏以下命令⾃动完成所有的编译：
```
./build.sh all # 只编译模块代码（u-Boot，kernel，Rootfs，Recovery）
# 需要再执⾏./mkfirmware.sh 进⾏固件打包
./build.sh # 编译模块代码（u-Boot，kernel，Rootfs，Recovery）
# 打包成update.img完整升级包
# 所有编译信息复制和⽣成到out⽬录下
```
默认是 Buildroot，可以通过设置坏境变量 RK_ROOTFS_SYSTEM 指定不同 rootfs。 

RK_ROOTFS_SYSTEM ⽬前可设定三种系统：buildroot、debian、 yocto 。

比如需要生成debian的命令如下：

```
export RK_ROOTFS_SYSTEM=debian
./build.sh
```

### 模块编译

```
./build.sh uboot
./build.sh kernel
./build.sh recovery
./build.sh rootfs
...
```


## 4、烧写固件

### 安装烧录工具

- Windows 驱动安装助手：```~/RK3588/linux-5.10-gen-rkr4/tools/windows/DriverAssitant_v5.12.zip```

- Windows 烧写⼯具：```~/RK3588/linux-5.10-gen-rkr4/tools/windows/RKDevTool_Release_v3.15```


### 打包工具

主要⽤于各分⽴固件打包成⼀个完整的update.img固件⽅便升级。

路径：```/tools/linux/Linux_Pack_Firmware/rockdev```

```
./mkupdate.sh
```


### 烧录

- 运行DriverAssitant_v5.12里面的DriverInstall.exe，先选择驱动卸载，然后再选择驱动安装。

- 通过Type-C数据线连接开发板与pc，运行RKDevTool.exe。若驱动安装没有问题，会自动识别到ADB设备，如图：
  

<img width="725" alt="烧录截图1" src="https://github.com/bolubolu/Linux-5.10-gen-rkr4/assets/54768057/8ce592cd-b227-4cfc-8ac4-7229b9a4024a">

注：若显示发现一个ADB设备，则在升级固件界面点击【切换】即可进入loader烧录模式。


- 按【固件】按钮，选择要升级的固件文件，加载固件之后，点击【升级】按钮，等待烧写完成即可。

<img width="723" alt="烧录截图2" src="https://github.com/bolubolu/Linux-5.10-gen-rkr4/assets/54768057/134f4b3f-fbb8-4c1d-be38-62200677a93a">

## 5、ADB使用

这里主要介绍windows下使用adb进行调试

步骤：
- 下载windows版本的adb.zip，解压到C:\adb
- 配置环境变量：
  
  1、键盘按键：win + r
  
  2、打开“系统属性”窗口
  
  3、“高级”→“环境变量”→“系统变量”
  
  4、找到“Path”双击，新建，复制adb路径进去，点击“确定”按钮，添加成功
  
- 常用的adb命令
```
  adb help            //可查看所有命令
  adb version
  adb start-server     //启动adb服务
  adb kill-server      //关闭adb服务
  adb devices
  adb shell
  adb push [-p] <local> <remote>
  adb pull [-p] [-a] <remote> [<local>]
```