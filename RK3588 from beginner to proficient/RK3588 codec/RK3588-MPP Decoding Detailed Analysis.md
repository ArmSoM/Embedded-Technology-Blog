# RK3588-MPP Decoding Detailed Analysis

# I. Introduction

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- This article makes a detailed analysis on RK3588 MPP decoding

# II. Environment Introduction

- Hardware environment: 
  ArmSoM-W3 RK3588 Development Board

- Software version:
  OS: ArmSoM-W3 Debian11

# III. Decoder Data Stream Interface
## 3.1 decode_put_packet
![](C:\Users\cherr\Desktop\编码1.png)

Input stream form: framed and non-framed  
MPP input is bare streams without encapsulation information. Bare stream input has two forms:  

1. Non-framed  
    This means the data has been segmented by frames, i.e. each MppPacket data input to decode_put_packet function already contains a complete frame, no more no less. In this case, MPP can process the stream directly by packets, which is the default running status of MPP.

2. Framed  
    Data read by length. Such data cannot determine whether an MppPacket data is a complete frame. Frame splitting is needed inside MPP. 

    MPP also supports such input, but the need_split flag inside MPP needs to be turned on through the MPP_DEC_SET_PARSER_SPLIT_MODE command of control interface before mpp_init.

    ```cpp
    	// NOTE: decoder split mode need to be set before init
        // framed input 
        RK_U32 need_split   = 1;
        mpi_cmd = MPP_DEC_SET_PARSER_SPLIT_MODE;
        param = &need_split;
        ret = mpi->control(ctx, mpi_cmd, param);
        if (MPP_OK != ret) {
            mpp_err("mpi->control failed\n");
            deInit(&packet, &frame, ctx, buf, data);
        }
    ```
    Thus the input MppPacket from decode_put_packet will be reframed by MPP, entering situation 1 processing.  

<font color="red" size="3">If the two situations are mixed, decoding error will occur.</font>

- Framed processing has higher efficiency but input stream needs parsing and framing first;
- Non-framed is simpler to use but less efficient.
- Non-framed is used in mpi_dec_test test case. Framed is used in Rockchip Android SDK. Users can choose both way based on application scenario and platform.



#  3.2 decode_get_frame 
![](C:\Users\cherr\Desktop\编码2.png)
## 3.3 Provide decoder sufficient memory space to save pixel data  
When decoding, <font color="red" size="3">decoder needs to get memory space to save output image pixel data.</font> Users need to provide decoder sufficient size. This space size requirement is calculated inside MPP decoder based on different chip platforms and video formats. The calculated memory requirement is provided to users via <font color="red" size="3">buf_size member variable of MppFrame</font>. <font color="red" size="3">Users need to allocate memory according to buf_size</font> to meet decoder requirement.

```cpp  
RK_U32 buf_size = mpp_frame_get_buf_size(frame); 

ret = mpp_buffer_group_limit_config(data->frm_grp, buf_size, 24);  
if (ret)  
{
    mpp_err("%p limit buffer group failed ret %d\n", ctx, ret);
    break;
}
```


## 3.4 Output image width/height info change 
When stream width, height, format, pixel bit depth etc change, it needs feedback to user. User needs to update decoder memory pool, and update new memory to decoder. This relates to decoder memory allocation and usage mode.

Image memory allocation and interaction modes:

> Mode 1: Pure Internal Allocation  
> Mode 2: Half Internal Allocation
> Mode 3: Pure External Allocation: Directly use external display memory, easy to achieve <font color="red" size="3">zero copy</font>.

### Mode 1: Pure Internal Allocation
Image memory allocated inside MPP decoder directly. Memory allocated by decoder itself. User gets decoder output image and releases after usage.  

In this mode, user does not need to call decoder control interface MPP_DEC_SET_EXT_BUF_GROUP command. 

Only need to call MPP_DEC_SET_INFO_CHANGE_READY command of control interface when decoder reports info change. 

Decoder will allocate memory automatically inside. 

User needs to release each frame data directly.

### Mode 2: Half Internal Allocation
User needs to create MppBufferGroup based on buf_size returned by get_frame MppFrame, and configure to decoder via control interface MPP_DEC_SET_EXT_BUF_GROUP. 

User can limit decoder memory usage through mpp_buffer_group_limit_config interface.  

### Mode 3: Pure External Allocation  
Create an empty external mode MppBufferGroup, and import memory block file handles allocated externally (usually dmabuf/ion/drm) from user. 

On Android platform, Mediaserver gets display memory from SurfaceFlinger via gralloc, and submits the file handles got by gralloc to MppBufferGroup.

Then configures MppBufferGroup to decoder via control interface MPP_DEC_SET_EXT_BUF_GROUP command. 

Then MPP decoder loops using the memory space got by gralloc.