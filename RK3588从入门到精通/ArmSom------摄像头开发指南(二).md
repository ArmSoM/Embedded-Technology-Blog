# 一. 简介

- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)

- 开发板：ArmSoM-W3

- Kernel：5.10.160

- OS：Debian11

  上篇文档介绍了rockchip平台怎么配置MIPI-CSI的通路，本⽂主要介绍在Rockchip平台下Camera相关测试命令





# 二. 摄像头连接

ArmSoM-W3开发板与imx415连接图如下:

<img src="C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ camera篇 ]--camera的开发指南\w3-camera-hardware.jpg" style="zoom: 67%;" />

注意

排线的金属引脚朝向板子





# 三. 使用摄像头

连接摄像头模块并上电后，可查看开机日志。

```
root@linaro-alip:/# dmesg | grep imx415
[    2.547754] imx415 3-001a: driver version: 00.01.08
[    2.547767] imx415 3-001a:  Get hdr mode failed! no hdr default
[    2.547819] imx415 3-001a: Failed to get power-gpios
[    2.547826] imx415 3-001a: could not get default pinstate
[    2.547831] imx415 3-001a: could not get sleep pinstate
[    2.547850] imx415 3-001a: supply dvdd not found, using dummy regulator
[    2.547918] imx415 3-001a: supply dovdd not found, using dummy regulator
[    2.547945] imx415 3-001a: supply avdd not found, using dummy regulator
[    2.613843] imx415 3-001a: Detected imx415 id 0000e0
[    2.613890] rockchip-csi2-dphy csi2-dphy0: dphy0 matches m00_b_imx415 3-001a:bus type 5
[   18.386174] imx415 3-001a: set fmt: cur_mode: 3864x2192, hdr: 0
[   18.389067] imx415 3-001a: set exposure(shr0) 2047 = cur_vts(2250) - val(203)
```

如果没有报错，那表明模块在正常运行，在Linux系统中，你可以使用多种方法来访问和利用该摄像头设备。





## 3.1 使用命令行工具

media-ctl 和 v4l2-ctl 是在Linux环境下用于配置和管理视频和多媒体设备的命令行工具。它们通常与V4L2（Video for Linux 2）子系统一起使用，用于管理摄像头、视频采集卡、显示设备和其他多媒体硬件的设置和参数。

media-ctl工具的操作是通过/dev/medio0等media 设备，它管理的是Media的拓扑结构中各个节点的
format、大小、 链接。
v4l2-ctl工具则是针对/dev/video0，/dev/video1等 video设备，它在video设备上进行set_fmt、
reqbuf、qbuf、dqbuf、stream_on、stream_off 等一系列操作。

```
 n为4的倍数(0,1,2,3…)
 /dev/videon+0：视频输出 SP主通道
 /dev/videon+1：视频输出 MP自身通道
 /dev/videon+2：3A统计
 /dev/videon+3：3A参数设置
```





### 3.1.1 显示拓扑结构

使用以下命令可以显示拓扑结构：

```
media-ctl -p -d /dev/media0
```

主要关注的是有没有找到Sensor的Entity。如果没有找到Sensor的Entity，说明Sensor注册有问题。

开发板上接上摄像头后可以看到如下的输出：

```
root@linaro-alip:# media-ctl -p -d /dev/media0
        ......
        ......
- entity 58: rockchip-csi2-dphy0 (2 pads, 2 links)
             type V4L2 subdev subtype Unknown flags 0
             device node name /dev/v4l-subdev1
	pad0: Sink
		[fmt:SGBRG10_1X10/3864x2192@10000/300000 field:none
		 crop.bounds:(12,16)/3840x2160]
		<- "m00_b_imx415 3-001a":0 [ENABLED]
	pad1: Source
		-> "rockchip-mipi-csi2":0 [ENABLED]

- entity 63: m00_b_imx415 3-001a (1 pad, 1 link)
             type V4L2 subdev subtype Sensor flags 0
             device node name /dev/v4l-subdev2
	pad0: Source
		[fmt:SGBRG10_1X10/3864x2192@10000/300000 field:none
		 crop.bounds:(12,16)/3840x2160]
		-> "rockchip-csi2-dphy0":0 [ENABLED]
```





### 3.2.2 查看摄像头信息

使用命令列出所有摄像头设备：

```
root@linaro-alip:/home/linaro# v4l2-ctl --list-devices
rk_hdmirx (fdee0000.hdmirx-controller):
	/dev/video20

rkisp-statistics (platform: rkisp):
	/dev/video18
	/dev/video19

rkcif-mipi-lvds2 (platform:rkcif):
	/dev/media0

rkcif (platform:rkcif-mipi-lvds2):
	/dev/video0
	/dev/video1
	/dev/video2
	/dev/video3
	/dev/video4
	/dev/video5
	/dev/video6
	/dev/video7
	/dev/video8
	/dev/video9
	/dev/video10

rkisp_mainpath (platform:rkisp0-vir0):
	/dev/video11
	/dev/video12
	/dev/video13
	/dev/video14
	/dev/video15
	/dev/video16
	/dev/video17
	/dev/media1
```

其中/dev/video11就是这个摄像头的设备。

查看设备的预览支持格式：

```
root@linaro-alip:/# v4l2-ctl --list-formats-ext --device /dev/video11
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture Multiplanar

        [0]: 'UYVY' (UYVY 4:2:2)
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
        [1]: 'NV16' (Y/CbCr 4:2:2)
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
        [2]: 'NV61' (Y/CrCb 4:2:2)
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
        [3]: 'NV21' (Y/CrCb 4:2:0)
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
        [4]: 'NV12' (Y/CbCr 4:2:0)
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
        [5]: 'NM21' (Y/CrCb 4:2:0 (N-C))
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
        [6]: 'NM12' (Y/CbCr 4:2:0 (N-C))
                Size: Stepwise 32x16 - 3840x2160 with step 8/8
```

查看设备的所有信息：

```
root@linaro-alip:/home/linaro# v4l2-ctl --all --device /dev/video11
Driver Info:
	Driver name      : rkisp_v6
	Card type        : rkisp_mainpath
	Bus info         : platform:rkisp0-vir0
	Driver version   : 2.0.0
	Capabilities     : 0x84201000
		Video Capture Multiplanar
		Streaming
		Extended Pix Format
		Device Capabilities
	Device Caps      : 0x04201000
		Video Capture Multiplanar
		Streaming
		Extended Pix Format
Media Driver Info:
	Driver name      : rkisp0-vir0
	Model            : rkisp0
	Serial           : 
	Bus info         : 
	Media version    : 5.10.110
	Hardware revision: 0x00000000 (0)
	Driver version   : 5.10.110
Interface Info:
	ID               : 0x03000007
	Type             : V4L Video
Entity Info:
	ID               : 0x00000006 (6)
	Name             : rkisp_mainpath
	Function         : V4L2 I/O
	Pad 0x01000009   : 0: Sink
	  Link 0x0200000a: from remote pad 0x1000004 of entity 'rkisp-isp-subdev': Data, Enabled
Priority: 2
Format Video Capture Multiplanar:
	Width/Height      : 3840/2160
	Pixel Format      : 'NV12' (Y/CbCr 4:2:0)
	Field             : None
	Number of planes  : 1
	Flags             : 
	Colorspace        : Default
	Transfer Function : Default
	YCbCr/HSV Encoding: Default
	Quantization      : Full Range
	Plane 0           :
	   Bytes per Line : 3840
	   Size Image     : 12441600
Selection Video Capture: crop, Left 0, Top 0, Width 3840, Height 2160, Flags: 
Selection Video Capture: crop_bounds, Left 0, Top 0, Width 3840, Height 2160, Flags: 
Selection Video Output: crop, Left 0, Top 0, Width 3840, Height 2160, Flags: 
Selection Video Output: crop_bounds, Left 0, Top 0, Width 3840, Height 2160, Flags: 

Image Processing Controls

                     pixel_rate 0x009f0902 (int64)  : min=0 max=1000000000 step=1 default=1000000000 value=356800000 flags=read-only, volatile
```





### 3.2.3 显示图像

使用v4l2-ctl抓一帧图片：

```
root@linaro-alip:/# v4l2-ctl --verbose -d /dev/video11 --set-fmt-video=width=3840,height=2160,pixelformat='NV12' --stream-mmap=4 --stream-skip=3 --stream-to=/data/4k_nv12.yuv --stream-count=4 --stream-poll
VIDIOC_QUERYCAP: ok
VIDIOC_G_FMT: ok
VIDIOC_S_FMT: ok
Format Video Capture Multiplanar:
        Width/Height      : 3840/2160
        Pixel Format      : 'NV12' (Y/CbCr 4:2:0)
        Field             : None
        Number of planes  : 1
        Flags             :
        Colorspace        : Default
        Transfer Function : Default
        YCbCr/HSV Encoding: Default
        Quantization      : Full Range
        Plane 0           :
           Bytes per Line : 3840
           Size Image     : 12441600
                VIDIOC_REQBUFS returned 0 (Success)
                VIDIOC_QUERYBUF returned 0 (Success)
                VIDIOC_QUERYBUF returned 0 (Success)
                VIDIOC_QUERYBUF returned 0 (Success)
                VIDIOC_QUERYBUF returned 0 (Success)
                VIDIOC_QBUF returned 0 (Success)
                VIDIOC_QBUF returned 0 (Success)
                VIDIOC_QBUF returned 0 (Success)
                VIDIOC_QBUF returned 0 (Success)
                VIDIOC_STREAMON returned 0 (Success)
cap dqbuf: 0 seq:      0 bytesused: 12441600 ts: 4953.039121 delta: 4953039.121 ms (ts-monotonic, ts-src-eof)
cap dqbuf: 1 seq:      1 bytesused: 12441600 ts: 4953.073968 (ts-monotonic, ts-src-eof)
cap dqbuf: 2 seq:      2 bytesused: 12441600 ts: 4953.106558 (ts-monotonic, ts-src-eof)
cap dqbuf: 3 seq:      3 bytesused: 12441600 ts: 4953.139114 (ts-monotonic, ts-src-eof)
cap dqbuf: 0 seq:      4 bytesused: 12441600 ts: 4953.173258 fps: 29.82 (ts-monotonic, ts-src-eof)
cap dqbuf: 1 seq:      5 bytesused: 12441600 ts: 4953.206177 fps: 29.93 (ts-monotonic, ts-src-eof)
cap dqbuf: 2 seq:      6 bytesused: 12441600 ts: 4953.239030 fps: 30.01 (ts-monotonic, ts-src-eof)
```

参数说明：

> -d： 摄像头对应设备文件
> --set-fmt-video：指定了宽高及pxielformat(用FourCC表示)。NV12即用FourCC表示的pixelformat
> --stream-mmap：指定buffer的类型为mmap，即由kernel分配的物理连续的或经过iommu映射的buffer
> --stream-to：指定帧数据保存的文件路径
> --stream-skip：指定丢弃(不保存到文件)前3帧
> --stream-count：指定抓取的帧数，不包括--stream-skip丢弃的数量



抓取的图片使用adb工具拷贝到Windows下用7YUV工具打开，也可以用ffplay命令打开

ffplay是FFmpeg提供的一个极为简单的音视频媒体播放器（由ffmpeg库和SDL库开发），可以用于音视频播放、可视化分析 ，提供音视频显示和播放相关的图像信息、音频的波形等信息，也可以用作FFmpeg API的测试工具使用。

使用 ffplay 非常简单，只需在终端中运行以下命令来播放媒体文件：

```
ffplay /data/4k_nv12.yuv -f rawvideo -pixel_format nv12 -video_size 3840x2160
```





### 3.2.3 显示视频

使用v4l2可以录制视频：

```
v4l2-ctl --verbose -d /dev/video11 --set-fmt-video=width=3840,height=2160,pixelformat='NV12' --stream-mmap=4 --set-selection=target=crop,flags=0,top=0,left=0,width=3840,height=2160 --stream-to=out.yuv
```

使用ffplay播放：

```
ffplay -f rawvideo -video_size 3840x2160 -pixel_format nv12 out.yuv
```





## 3.3 使用多媒体框架应用程序

**GStreamer**：GStreamer是一种多媒体框架，你可以使用它来构建自定义的多媒体应用程序，捕获摄像头视频，进行处理和展示。

你可以使用以下GStreamer管道捕获视频：

```
gst-launch-1.0 v4l2src device=/dev/video11 ! video/x-raw,format=NV12,width=3840,height=2160,framerate=30/1 ! videoconvert ! autovideosink
```

- v4l2src：从 /dev/video11 捕获视频数据。
- video/x-raw：指定输出数据格式为原始视频，format 参数设置为 NV12，width 设置为 3840，height 设置为 2160，framerate 设置为 30fps。
- videoconvert：执行格式转换，确保输出数据适用于后续的元素。
- autovideosink：自动选择适当的视频输出插件，将视频显示在屏幕上。

显示如下：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588平台驱动调试篇[ camera篇 ]--camera的开发指南\camera-screenshot.png)

注意：显示器的分辨率是1920x1080，摄像头的分辨率是3840x2160，导致如左上角画面显示不全，双击边框可以得到分辨率是1920x1080的画面。



下面有一个简单的Python示例，用于创建一个GStreamer管道并启动捕获视频可供参考：

```
import gi
gi.require_version('Gst', '1.0')
from gi.repository import Gst

Gst.init(None)

# 创建一个新的GStreamer管道
pipeline = Gst.Pipeline()

# 创建元素，设置摄像头设备和分辨率
src = Gst.ElementFactory.make("v4l2src", "v4l2-source")
src.set_property("device", "/dev/video11")

caps = Gst.Caps.from_string("video/x-raw,format=NV12,width=3840,height=2160,framerate=30/1")
filter = Gst.ElementFactory.make("capsfilter", "filter")
filter.set_property("caps", caps)

convert = Gst.ElementFactory.make("videoconvert", "converter")
sink = Gst.ElementFactory.make("autovideosink", "videosink")

# 将元素添加到管道中
pipeline.add(src)
pipeline.add(filter)
pipeline.add(convert)
pipeline.add(sink)

# 链接元素
src.link(filter)
filter.link(convert)
convert.link(sink)

# 设置管道的状态为播放
pipeline.set_state(Gst.State.PLAYING)

try:
    # 在此处添加自定义的处理逻辑
    # 你可以添加信号处理、保存视频到文件、处理图像等操作
    pass
except KeyboardInterrupt:
    pass

# 停止管道并释放资源
pipeline.set_state(Gst.State.NULL)
```





## 3.4.自定义应用程序开发

特定的定制功能，一般是使用编程语言（如C++或Python）开发自己的摄像头应用程序

总的来说，应用程序通过API接口采集视频数据大致分为五个步骤：

首先，打开视频设备文件，进行视频采集的参数初始化，设置视频图像的采集窗口、采集的点阵大小和格式;

其次，申请若干视频采集的帧缓冲区，并将这些帧缓冲区从内核空间映射到用户空间，便于应用程序读取/处理视频数据;

第三，将申请到的帧缓冲区在视频采集输入队列排队，并启动视频采集;

第四，驱动开始视频数据的采集，应用程序从视频采集输出队列取出帧缓冲区，处理完后，将帧缓冲区重新放入视频采集输入队列，循环往复采集连续的视频数据;

第五，停止视频采集。





# 四. 结语

根据需求，选择最适合的方法来访问和使用摄像头设备，每个具体型号的摄像头可能有其独特的设置和要求，各个系统下的使用摄像头的方法也有很多，如果你有疑问或者需要帮助，可以在[ArmSom论坛](http://forum.armsom.org/)提出问题，与其他开发者分享经验和获取支持。







