# 1. 简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 场景：在RK3588上做qt开发工作

- RK3588安装Qt+opencv+采集摄像头画面

# 2. 环境介绍


- 硬件环境：
ArmSoM-W3 RK3588开发板、MIPI-CSI摄像头( ArmSoM官方配件 )

- 软件版本：
OS：ArmSoM-W3 Debian11
QT：QT5.15.2（Qt Creator：4.11.0）
OpenCV：3.4.14

![在这里插入图片描述](https://img-blog.csdnimg.cn/a718412de3264243857662fe414fffac.png#pic_center =600x)

# 3. 在RK3588上安装QT
- RK3588开发板联网，确认好是否能访问网络
- sudu su切换到root用户
- 安装交叉编译
	```cpp
	sudo su
	
	sudo apt-get update
	
	sudo apt-get upgrade
	
	sudo apt-get install build-essential
	```

- 使用如下步骤安装Qt

	```cpp
	sudo apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
	sudo apt-get install qtcreator
	sudo apt-get install qt5*
	```
- 安装完成后在应用程序中搜索Qt会有Qt的相关程序。
- 执行命令：qmake -query 查看RK3588上安装的QT版本：QT5.15.2

	```bash
	qmake -query
	```

# 4. 在RK3588上安装opencv

## 4.1 安装依赖

```cpp
$sudo apt-get install cmake
$sudo apt-get install gcc g++
$sudo apt-get install python3-dev python3-numpy
$sudo apt-get install build-essential
$sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev libv4l-dev libxvidcore-dev libx264-dev libatlas-base-dev gfortran libgtk2.0-dev libjpeg-dev libpng-dev
```
## 4.2 下载OpenCV
- 到 [https://codeload.github.com/opencv/opencv/zip/refs/tags/3.4.14](https://codeload.github.com/opencv/opencv/zip/refs/tags/3.4.14)下载OpenCV。
- 解压opencv-3.4.14.zip压缩包会生成opencv-3.4.14文件夹。

## 4.3 编译安装OpenCV
- 进入opencv-3.4.14文件夹，在该目录下新建一个build文件夹；执行如下命令：

	```cpp
	cd opencv-3.4.14
	mkdir build
	cd build 
	cmake ../
	make		#执行make开始编译，这个时间比较长，耐心等待。 
	sudo make install
	```

## 4.4 配置动态链接库
- 编辑/etc/ld.so.conf，文末加入“/usr/local/lib”，执行sudo /sbin/ldconfig -v生效。
	```cpp
	sudo /sbin/ldconfig -v
	```



# 5. QT开发
## 5.1 创建qt项目
- 进入qt创建一个新Qt项目，创建完后打开.pro文件，加入opencv库的路径。

	```cpp
	INCLUDEPATH +=/usr/local/include/ \
	              /usr/local/include/opencv/ \
	              /usr/local/include/opencv2
	LIBS +=/usr/local/lib/lib*
	```
- 可以通过v4l2-ctl --list-devices来获取摄像头的节点。
	```
	root@linaro-alip:~# v4l2-ctl --list-devices
	rk_hdmirx (fdee0000.hdmirx-controller):
	    /dev/video20
	
	
	rkisp-statistics (platform: rkisp):
	    /dev/video18
	    /dev/video19
	
	
	rkcif-mipi-lvds2 (platform:rkcif):
	    /dev/media0
	
	
	rkisp_mainpath (platform:rkisp0-vir0):
	    /dev/video11
	    /dev/video12
	    /dev/video13
	    /dev/video14
	    /dev/video15
	    /dev/video16
	    /dev/video17
	    /dev/media1
	
	罗技高清网络摄像机 C93 (usb-fc880000.usb-1):
	    /dev/video21
	    /dev/video22
	    /dev/media2
	```
## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)
