# RK3588开发板 (armsom-w3) 之 USB摄像头图像预览

## 硬件准备



RK3588开发板（armsom-w3）、USB摄像头（罗技高清网络摄像机 C93）、1000M光纤 、 串口调试工具

![image-20230802163204676](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230802163204676.png)

## v4l2采集画面

v4l2-ctl是一个用于Linux系统的命令行实用程序，用于控制视频4 Linux 2（V4L2）设备。V4L2是Linux内核中的视频设备驱动框架，用于支持各种摄像头、摄像头和视频采集设备。

将USB摄像头插入开发板后，会有如下打印：

```
[14720.842825] usb 7-1: new low-speed USB device number 2 using xhci-hcd
[14720.986597] usb 7-1: New USB device found, idVendor=413c, idProduct=301a, bcdDevice= 1.00
[14720.986638] usb 7-1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[14720.986653] usb 7-1: Product: Dell MS116 USB Optical Mouse
[14720.986667] usb 7-1: Manufacturer: PixArt
[14721.008123] input: PixArt Dell MS116 USB Optical Mouse as /devices/platform/usbdrd3_1/fc400000.usb/xhci-hcd.5.auto/usb7/7-1/7-1:1.0/0003:413C:301A.0001/input/input5
```

使用v4l2-ctl --list-devices来获取usb摄像头的节点：

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

运行 v4l2-ctl -d /dev/video21 --list-formats-ext 命令可以查看你的摄像头支持的格式：



![Image](C:\Users\Administrator\AppData\Local\Temp\Image.png)

![image-20230802164835405](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230802164835405.png)

从这个命令可以看到这款摄像头支持两种格式“YUYV” "MJPG".

### 摄像头YUYV格式画面采集

介绍一些常用的v4l2-ctl命令选项和功能

```
v4l2-ctl -d /dev/video21 -D // 确认video节点

v4l2-ctl -d /dev/video21 --get-fmt-video // 确认分辨率和图像格式

v4l2-ctl -d /dev/video21 --get-dv-timings //获取当前timings

v4l2-ctl -d /dev/video21 --query-dv-timings // 实时查询timings
```

查看当前参数：使用v4l2-ctl -d /dev/video21 -l命令，将显示当前连接到/dev/video21设备的所有控制参数和其当前值。

设置视频格式和帧率：通过指定视频格式和帧率来配置摄像头，例如：

```
v4l2-ctl -d /dev/video0 --set-fmt-video=width=1280,height=720,pixelformat=YUYV
v4l2-ctl -d /dev/video0 --set-parm=30
```

这里使用如下命令采集一帧画面报错到/data/01.yuv

```
v4l2-ctl --verbose -d /dev/video21 --set-fmt-video=width=640,height=480,pixelformat=YUYV --stream-mmap=4 --stream-skip=3 --stream-count=5 --stream-to=/data/01.yuv --stream-poll
```

将文件通过使用adb上传到PC端使用7YUV工具查看

![Image](C:\Users\Administrator\AppData\Local\Temp\Image.png)

### 摄像头MJPG格式画面采集

`mjpg-streamer` 是 [github](https://github.com/andyshrk/mjpg-streamer)上一个开源的 uvc 视频应用，它可以获取摄像头的视频流，然后通过局域网传输，可以直接在armsom-w3开发板上编译这个代码并运行：

```
git clone https://github.com/andyshrk/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental/
apt install cmake libjpeg62-turbo-dev build-essential
make
```

在编译过程中，遇到两个报错问题，防止后续遇上再找资料，这里也附上：

> Scanning dependencies of target input_opencv
> make[3]: 离开目录“/root/mjpg-streamer/mjpg-streamer-experimental/_build”
> make[3]: 进入目录“/root/mjpg-streamer/mjpg-streamer-experimental/_build”
> [ 34%] Building CXX object plugins/input_opencv/CMakeFiles/input_opencv.dir/input_opencv.cpp.o
> /root/mjpg-streamer/mjpg-streamer-experimental/plugins/input_opencv/input_opencv.cpp:86:5: warning: invalid suffix on literal; C++11 requires a space between literal and string macro [-Wliteral-suffix]
>    86 |     " Help for input plugin..: "INPUT_PLUGIN_NAME"\n" \
>       |     ^
> /root/mjpg-streamer/mjpg-streamer-experimental/plugins/input_opencv/input_opencv.cpp: In function ‘void* worker_thread(void*)’:
> /root/mjpg-streamer/mjpg-streamer-experimental/plugins/input_opencv/input_opencv.cpp:408:34: error: ‘CV_IMWRITE_JPEG_QUALITY’ was not declared in this scope; did you mean ‘IN_CMD_JPEG_QUALITY’?
>   408 |     compression_params.push_back(CV_IMWRITE_JPEG_QUALITY);
>       |                                  ^~~~~~~~~~~~~~~~~~~~~~~
>       |                                  IN_CMD_JPEG_QUALITY
> make[3]: *** [plugins/input_opencv/CMakeFiles/input_opencv.dir/build.make:82：plugins/input_opencv/CMakeFiles/input_opencv.dir/input_opencv.cpp.o] 错误 1
> make[3]: 离开目录“/root/mjpg-streamer/mjpg-streamer-experimental/_build”
> make[2]: *** [CMakeFiles/Makefile2:468：plugins/input_opencv/CMakeFiles/input_opencv.dir/all] 错误 2
> make[2]: 离开目录“/root/mjpg-streamer/mjpg-streamer-experimental/_build”
> make[1]: *** [Makefile:149：all] 错误 2
> make[1]: 离开目录“/root/mjpg-streamer/mjpg-streamer-experimental/_build”
> make: *** [Makefile:19：all] 错误 2
> root@linaro-alip:~/mjpg-streamer/mjpg-streamer-experimental# 

1.关于无效后缀的警告： 找到 "/root/mjpg-streamer/mjpg-streamer-experimental/plugins/input_opencv/input_opencv.cpp" 文件的第86行，并确保在`" Help for input plugin..: "INPUT_PLUGIN_NAME"\n"`中的字符串宏`INPUT_PLUGIN_NAME`之前有一个空格。

2.关于 'CV_IMWRITE_JPEG_QUALITY' 未声明的错误： 找到 "/root/mjpg-streamer/mjpg-streamer-experimental/plugins/input_opencv/input_opencv.cpp" 文件的第408行，并将 'CV_IMWRITE_JPEG_QUALITY' 替换为 'IN_CMD_JPEG_QUALITY'。



编译成功之后执行如下命令验证

```
./mjpg_streamer -i "./input_uvc.so -n -f 30 -r 640x480 -d /dev/video21" -o "./output_http.so -w ./www" &
```

![Image](C:\Users\Administrator\AppData\Local\Temp\Image.png)




