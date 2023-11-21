# 简介
- 学习完MPP的解码Demo之后，想必大家都想通过一个项目来进行RK3588-MPP的解码实战。
- 本篇文章就基于ArmSoM-W3开发板，开发一个多路硬解码项目，实现四路MPP硬解码拉流显示
- 实现的效果如下：

[video(video-bULbEpcZ-1698739904454)(type-csdn)(url-https://live.csdn.net/v/embed/340015)(image-https://video-community.csdnimg.cn/vod-84deb4/43a2282077c471ee84724531958c0102/snapshots/dbec80cfc3b649f28c68dd35a0200188-00003.jpg?auth_key=4852339544-0-0-fb40df888cbe009bd94eca12dd878154)(title-RK3588四路MPP硬解码拉流)]


# 环境介绍

- 硬件环境：
ArmSoM-W3 RK3588开发板

- 软件版本：
OS：ArmSoM-W3 Debian11

# 思路：
### ArmSoM-W3 + QT+FFmpeg +RTSP+ MPP实现多路硬解码拉流

- mpp对外接口是输入MppPacket结构体指针：MppPacket *

	那么，MppPacket 数据从哪里来？
- <font color="red" size="3"> 通过FFmpeg进行拉流，拉RTSP流解封装为AVPacket数据类型，然后传给mpp进行硬解码</font>

	```bash
	 1.首先由ffmpeg完成拉流工作获取到AVPacket。
	
	 2.MPP接收到AVPacket数据然后转换成MppPacket再进行硬解码。
	
	 3.MPP解码之后交给rga负责图片格式转换裁切等工作。
	
	 4.交给qt渲染显示。
	```

## 1. FFmpeg打开MP4格式文件或者进行拉流获取到AVPacket 
**核心代码：**

```cpp
AVPacket *av_packet = nullptr;
av_packet = (AVPacket *)av_malloc(sizeof(AVPacket));

char filepath[] = "rtsp://admin:armsom@80.0.0.211:854/armsomvideo";// rtsp 地址

avformat_open_input(&pFormatCtx, filepath, nullptr, &options)    //打开多媒体流，并且获取一些信息

 //读取一帧数据存到av_packet，av_packet是FFmpeg和MMP的数据互通接口
av_read_frame(pFormatCtx, av_packet) 
```

## 2. MPP获取到从FFmpeg传过来的AVPacket 数据然后进行硬解码
**核心代码：**

```cpp
//将FFmpeg拉流获取到的av_packet数据通过函数参数传给MPP进行硬解码
int MppDecode::decode_simple(MppDecode::MpiDecLoopData *data, AVPacket *av_packet)
{
	MPP_RET ret = MPP_OK;
	MppPacket packet = nullptr;
    MppFrame  frame  = nullptr;

   //将AVPacket 数据转换为MppPacket数据 (实际上是MppPacket  ->data =  AVPacket  ->data)
    ret = mpp_packet_init(&packet, av_packet->data, av_packet->size); 
	mpp_packet_set_pts(packet, av_packet->pts);
	mpp_packet_set_dts(packet, av_packet->dts);
	
	// 输入MppPacket，输出MppFrame
	mpi->decode_put_packet(ctx, packet)
	mpi->decode_get_frame(ctx, &frame)
}
```


## 3. MPP解码之后交给rga负责图片格式转换裁切等工作
- 从MPP解码后获得的数据是YUV_420sp类型。我们用RGA将其转换成RGB888的数据数据格式方便QT显示
## 4. qt渲染显示
- qt渲染方面可以通过lable或者OpenGL来显示



## 5. 更多项目设计详情请前往ArmSoM官方论坛进行讨论
### ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
### ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)