# ArmSoM-W3(RK3588):MPP Implements Multi-Channel Hardware Decoding Streaming

# Introduction

- After learning the MPP decoding demos, you must be eager to have a project for RK3588 MPP decoding practice.  
- This article develops a multi-channel hardware decoding project on the ArmSoM-W3 board to implement four-channel MPP hardware decoding streaming display.
- The implemented effect is as follows:

[video(video-bULbEpcZ-1698739904454)(type-csdn)(url-https://live.csdn.net/v/embed/340015)(image-https://video-community.csdnimg.cn/vod-84deb4/43a2282077c471ee84724531958c0102/snapshots/dbec80cfc3b649f28c68dd35a0200188-00003.jpg?auth_key=4852339544-0-0-fb40df888cbe009bd94eca12dd878154)(title-RK3588四路MPP硬解码拉流)]


# Environment Introduction

- Hardware environment: 
ArmSoM-W3 RK3588 Development Board

- Software version:  
OS: ArmSoM-W3 Debian11

# How to realize？
### ArmSoM-W3 + QT + FFmpeg + RTSP + MPP Multi-Channel Hardware Decoding Streaming

- The external interface of mpp is the pointer to MppPacket structure: MppPacket*

	So where does the MppPacket data come from?
	
- <font color="red" size="3"> Stream RTSP via FFmpeg, decapsulate it into AVPacket data type, then pass to mpp for hardware decoding</font>

	```bash
	 1. FFmpeg streams and gets AVPacket firstly.
	
	 2. MPP receives AVPacket data, and converts it to MppPacket, then hardware decodes.
	
	 3. After MPP decodes, RGA is responsible for image format conversion, cropping etc.
	
	 4. Pass to qt for rendering and display.
	```

## 1. FFmpeg opens MP4 file or streams to get AVPacket
**Core Code:**

```cpp
AVPacket *av_packet = nullptr;  
av_packet = (AVPacket *)av_malloc(sizeof(AVPacket));

char filepath[] = "rtsp://admin:armsom@80.0.0.211:854/armsomvideo"; //rtsp address

avformat_open_input(&pFormatCtx, filepath, nullptr, &options) //open stream, and get info

 //read 1 frame into av_packet，which is the data exchange interface between FFmpeg and MMP  
av_read_frame(pFormatCtx, av_packet)
```

## 2. MPP gets AVPacket data from FFmpeg then hardware decodes
**Core Code:** 

```cpp 
//pass av_packet data got from FFmpeg streaming to MPP for hardware decoding via function parameter
int MppDecode::decode_simple(MppDecode::MpiDecLoopData *data, AVPacket *av_packet)  
{   
	MPP_RET ret = MPP_OK;
	MppPacket packet = nullptr;
    MppFrame  frame  = nullptr;

   //convert AVPacket data to MppPacket (Actually MppPacket ->data = AVPacket ->data)
    ret = mpp_packet_init(&packet, av_packet->data, av_packet->size);
	mpp_packet_set_pts(packet, av_packet->pts); 
	mpp_packet_set_dts(packet, av_packet->dts);   
	
	// input MppPacket, output MppFrame
	mpi->decode_put_packet(ctx, packet)  
	mpi->decode_get_frame(ctx, &frame)
}
```


## 3. After MPP decodes, RGA handles image format conversion, cropping etc.  
- The data from MPP decoding is YUV_420sp type. We use RGA to convert it to RGB888 data format for convenient QT display.
## 4. QT rendering and display
- QT rendering can use label or OpenGL for display

## 5.Let's disscuss about more project design details on ArmSoM official forum
### ArmSoM Product Introduction: [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
### ArmSoM Technical Forum: [http://forum.armsom.org](http://forum.armsom.org)