# 一. 简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 本篇文章进行RK3588-MPP解码的详细解析
# 二. 环境介绍

- 硬件环境：
  ArmSoM-W3 RK3588开发板

- 软件版本：
  OS：ArmSoM-W3 Debian11

# 三. 解码器数据流接口
## 3.1 decode_put_packet
![在这里插入图片描述](https://img-blog.csdnimg.cn/dd39c493bfc64283926360024c75e66b.png)
输入码流的形式：分帧与不分帧
MPP 的输入都是没有封装信息的裸码流，裸码流输入有两种形式：
1. 不分帧
这种方式是已经按帧分段的数据，即每一包输入给 decode_put_packet 函数的 MppPacket 数据都已经包含完整的一帧，不多也不少。在这种情况下，MPP 可以直接按包处理码流，是 MPP 的默认运行情况。
2. 分帧
按长度读取的数据，这样的数据无法判断一包 MppPacket 数据是否是完整的一帧，需要
MPP 内部进行分帧处理。MPP 也可以支持这种形式的输入，但需要在 mpp_init 之前，通过 control
接口的 MPP_DEC_SET_PARSER_SPLIT_MODE 命令，MPP 内的 need_split 标志打开。

	```cpp
		// NOTE: decoder split mode need to be set before init
	    // 按帧输入码流
	    RK_U32 need_split   = 1;
	    mpi_cmd = MPP_DEC_SET_PARSER_SPLIT_MODE;
	    param = &need_split;
	    ret = mpi->control(ctx, mpi_cmd, param);
	    if (MPP_OK != ret) {
	        mpp_err("mpi->control failed\n");
	        deInit(&packet, &frame, ctx, buf, data);
	    }
	```
这样，调用 decode_put_packet 输入的 MppPacket 就会被 MPP 重新分帧，进入到情况一的处理。

<font color="red" size="3">如果这两种情况出现了混用，会出现码流解码出错的问题。

- 分帧方式处理效率高，但需要输入码流之前先进行解析与分帧；
- 不分帧方式使用简单，但效率会受影响。
- 在 mpi_dec_test 的测试用例中，使用的是方式不分帧的方式。在瑞芯微的 Android SDK 中，使用的是分帧的方式。用户可以根据自己的应用场景和平台条件进行选择



#  3.2 decode_get_frame
![在这里插入图片描述](https://img-blog.csdnimg.cn/8bcea2d2d28c4a2cad55c8fd10f462dc.png)
##  3.3 给解码器提供足够大小的保存像素数据的内存空间
解码器在解码时，<font color="red" size="3">需要为输出图像获取保存像素数据的内存空间，</font>用户需要给解码器提供足够大小，这个空间大小的需求，会在 MPP 解码器内部根据不同的芯片平台以及不同的视频格式需求进行计算，计算后的内存空间需求会通过<font color="red" size="3">MppFrame 的成员变量 buf_size</font> 提供给用户。<font color="red" size="3">用户需要按 buf_size的大小进行内存分配，</font>即可满足解码器的要求。

```cpp
RK_U32 buf_size = mpp_frame_get_buf_size(frame);

ret = mpp_buffer_group_limit_config(data->frm_grp, buf_size, 24);
if (ret) 
{
    mpp_err("%p limit buffer group failed ret %d\n", ctx, ret);
    break;
}
```


## 3.4 输出图像的变宽高信息（Info change）
当码流的宽高，格式，像素位深等信息发生变化时，需要反馈给用户，用户需要更新解码器使用的
内存池，把新的内存更新给解码器。这里涉及到解码内存分配与使用模式。
图像内存分配以及交互模式: 

> 模式一：纯内部分配模式 
> 模式二：半内部分配模式 
> 模式三：纯外部分配模式: 直接使用外部显示用的内存，容易实现<font
> color="red" size="3">零拷贝。</font>
>
### 模式一：纯内部分配模式
图像内存直接从 MPP 解码器内部分配，内存由解码器直接分配，用户得到解码器输出图像，在使用
完成之后直接释放。
在这种方式下，用户不需要调用解码器 control 接口的 MPP_DEC_SET_EXT_BUF_GROUP 命令，只
需要在解码器上报 info change 时直接调用 control 接口的 MPP_DEC_SET_INFO_CHANGE_READY
命令即可。解码器会自动在内部进行内存分配，用户需要把获取到的每帧数据直接释放。
### 模式二：半内部分配模式 
用户需要根据get_frame返回的MppFrame的buf_size
来创建 MppBufferGroup，并通过 control 接口的 MPP_DEC_SET_EXT_BUF_GROUP 配置给解码器。用户可以通过 mpp_buffer_group_limit_config 接口来限制解码器的内存使用量。

### 模式三：纯外部分配模式
这种模式通过创建空的 external 模式的 MppBufferGroup，从用户那里导入外部分配器分析的内存块
文件句柄（一般是 dmabuf/ion/drm）。在 Android 平台上，Mediaserver 通过 gralloc 从 SurfaceFlinger
获取显示用内存，把 gralloc 得到的文件句柄提交（commit）到 MppBufferGroup 里，再把
MppBufferGroup 通过 control 接口 MPP_DEC_SET_EXT_BUF_GROUP 命令配置给解码器，然后 MPP
解码器将循环使用 gralloc 得到的内存空间。