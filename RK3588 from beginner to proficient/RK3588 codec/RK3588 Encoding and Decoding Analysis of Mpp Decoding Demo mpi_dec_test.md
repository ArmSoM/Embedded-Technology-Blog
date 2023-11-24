# RK3588 Encoding and Decoding Analysis of Mpp Decoding Demo: mpi_dec_test

# 1. Introduction

- mpi_dec_test is the official decoding demo from Rockchip
- This article analyzes the code and decoding flow of mpi_dec_test

# 2. Environment Introduction  

- Hardware environment:
  ArmSoM-W3 RK3588 Development Board

- Software version: 
  OS: ArmSoM-W3 Debian11

# 3. Mpp Decoding Flow Analysis  



![image](https://img-blog.csdnimg.cn/27dfa966953446019431ca9f7e5b7e62.png#pic_center)


- mpp_create: get MppCtx instance and MppApi structure
- mpp_init: init decoding type and format of MppCtx  
- mpi->control: configure decoding parameters through commands
- decode_put_packet: input stream: encoded data MppPacke, e.g. H264, H265 data
- decode_get_frame: get decoded data stored in MppFrame, e.g. YUV, RGB data 
- mpi->reset: reset decoder to normal initialized status
- mpp_destroy: release allocated memory, destroy properly   

# 4. Important Function Analysis   
## mpp_init function: init decoding type and format of MppCtx  

**Prototype of mpp_init function:**  

```cpp
MPP_RET mpp_init(MppCtx ctx, MppCtxType type, MppCodingType coding)
```
**Instance of calling mpp_init function:**   

```cpp 
ret = mpp_init(ctx, MPP_CTX_DEC, MppCodingType::MPP_VIDEO_CodingAVC);  
if (ret)  
 {
    mpp_err("mpp_init failed ret %d\n", ret);    
    goto MPP_TEST_OUT;  
 } 
```

**Parameter analysis of mpp_init function:**
- MppCtxType parameter: init encoder or decoder

	```cpp
	MPP_CTX_DEC : decoder   
	MPP_CTX_ENC : encoder 
	```

- MppCodingType parameter: coding format

	```cpp   
	MPP_VIDEO_CodingAVC : H.264      
	MPP_VIDEO_CodingHEVC: H.265  
	MPP_VIDEO_CodingVP8 : VP8  
	MPP_VIDEO_CodingVP9 : VP9     
	MPP_VIDEO_CodingMJPEG : MJPEG  
	```

# 5. mpi_dec_test Flow Analysis
**mpi_dec_test decode command example:**   

```bash   
sudo mpi_dec_test -i /oem/200frames_count.h264 -t 7 -n 200 -o /oem/decode.yuv -w 1920 -h 1080  
```

**mpi_dec_test flow analysis:**  

```bash 
main() ---> dec_decode() ---> thread_decode() ---> dec_simple() ---> mpp_destroy()   
```

- The main function parses parameters based on input cmd params <font color="red" size="3">(cmd param char **argv corresponds to command -i /oem/200frames_count.h264 -t 7 -n 200 -o /oem/decode.yuv -w 1920 -h 1080)</font> and saves to structure <font color="red" size="3">MpiDecTestCmd *cmd</font>  
-  <font color="red" size="3">dec_decode(cmd)</font> is the encapsulated decoding function. Pass in structure <font color="red" size="3">MpiDecTestCmd * cmd</font> to complete decoding  
- Function <font color="red" size="3">dec_decode </font>executes some MPP init operations: mpp_create() mpp_init(), mpp_dec_cfg_init(), mpi->control. After init, create decode thread: <font color="red" size="3">thread_decode</font> to decode.   
- Decode thread: <font color="red" size="3">thread_decode</font> thread judges whether to use <font color="red" size="3">dec_simple </font>decoding or <font color="red" size="3">dec_advanced </font>decoding based on variable <font color="red" size="3">cmd->simple</font>    
- After decoding the last frame, execute <font color="red" size="3">pthread_join()</font> function to wait for thread_decode： <font color="red" size="3">thread_decode</font> to end and release thread  
- After thread release, execute <font color="red" size="3">reset</font> : <font color="red" size="3">mpi->reset(ctx)</font> reset decoder to normal initialized status.    
- After decoder reset, <font color="red" size="3">release allocated memory through mpp_destroy()</font>, some destroy operations to prevent memory leak.  

# 6. mpi_dec_test Usage Example  

Execute decode command in terminal:  
```bash   
sudo mpi_dec_test -i /oem/200frames_count.h264 -t 7 -n 200 -o /oem/decode.yuv -w 1920 -h 1080 
```
- Analysis of command: -t 7: input H.264 stream, -i :input file, -n 200 :decode 200 frames, -w :image width, -h :image height  

Decode output as below:   

```bash
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: input file /oem/200frames_count.h264 size 87402
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: cmd parse result:
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: input  file name: /oem/200frames_count.h264
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: output file name: /oem/decode.yuv
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: width      : 1920
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: height     : 1080
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: type       :    7
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_utils: max frames :  200
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: mpi_dec_test start
Oct 18 12:26:06 linaro-alip mpp[2150]: mpp_info: mpp version: 8a54ab8d author: xueman.ruan   2023-06-16 [h264d]: fix the derivation of mbaff.
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 mpi_dec_test decoder test start w 1920 h 1080 type 7
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode_get_frame get info changed found
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decoder require buffer w:h [640:480] stride [640:480] buf_size 614400
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 0
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 1
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 2
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 3
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 4
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 5
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 6
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 7
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 8
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 9
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 10
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 11
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 12
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 13
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 14
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 15
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 16
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 17
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 18
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 19
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 20
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 21
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 22
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 23
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 24
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 25
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 26
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 27
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 28
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 29
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 30
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 31
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 32
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 33
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 34
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 35
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 36
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 37
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 38
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 39
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 40
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 41
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 42
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 43
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 44
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 45
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 46
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 47
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 48
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 49
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 50
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 51
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 52
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 53
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 54
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 55
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 56
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 57
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 58
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 59
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 60
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 61
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 62
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 63
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 64
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 65
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 66
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 67
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 68
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 69
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 70
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 71
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 72
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 73
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 74
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 75
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 76
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 77
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 78
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 79
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 80
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 81
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 82
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 83
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 84
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 85
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 86
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 87
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 88
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 89
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 90
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 91
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 92
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 93
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 94
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 95
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 96
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 97
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 98
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 99
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 100
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 101
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 102
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 103
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 104
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 105
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 106
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 107
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 108
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 109
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 110
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 111
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 112
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 113
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 114
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 115
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 116
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 117
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 118
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 119
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 120
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 121
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 122
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 123
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 124
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 125
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 126
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 127
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 128
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 129
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 130
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 131
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 132
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 133
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 134
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 135
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 136
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 137
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 138
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 139
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 140
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 141
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 142
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 143
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 144
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 145
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 146
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 147
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 148
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 149
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 150
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 151
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 152
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 153
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 154
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 155
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 156
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 157
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 158
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 159
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 160
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 161
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 162
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 163
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 164
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 165
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 166
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 167
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 168
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 169
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 170
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 171
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 172
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 173
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 174
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 175
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 176
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 177
Oct 18 12:26:06 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 178
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 179
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 180
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 181
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 182
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 183
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 184
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 185
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 186
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 187
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 188
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 189
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 190
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 191
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 192
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 193
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 194
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 195
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 loop again
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 196
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 197
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 198
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: 0x55aceed120 decode get frame 199
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: decode 200 frames time 328 ms delay   2 ms fps 609.30
Oct 18 12:26:07 linaro-alip mpp[2150]: mpi_dec_test: test success max memory 5.86 MB


```