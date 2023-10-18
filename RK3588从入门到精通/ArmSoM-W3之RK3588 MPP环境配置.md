# 1. 简介
瑞芯微提供的媒体处理软件平台（Media Process Platform，简称 MPP）是适用于瑞芯微芯片系列的
通用媒体处理软件平台。该平台对应用软件屏蔽了芯片相关的复杂底层处理，其目的是为了屏蔽不
同芯片的差异，为使用者提供统一的视频媒体处理接口（Media Process Interface，缩写 MPI）。MPP
提供的功能包括：
- 视频解码
H.265 / H.264 / H.263 / VP9 / VP8 / MPEG-4 / MPEG-2 / MPEG-1 / VC1 / MJPEG

- 视频编码
H.264 / VP8 / MJPEG

- 视频处理

- 视频拷贝，缩放，色彩空间转换，场视频解交织（Deinterlace）

# 2. 环境介绍

- 硬件环境：
ArmSoM-W3 RK3588开发板

- 软件版本：
OS：ArmSoM-W3 Debian11
# 3. RK3588 MPP环境配置

## 3.1. 下载安装rkmpp
- 从github下载mpp包：

	```bash
	git clone https://github.com/rockchip-linux/mpp.git
	```

- 编译安装

	```bash
	cd mpp/build/linux/aarch64 
	 
	./make-Makefiles.bash
	
	make -j8
	
	sudo make install     
	```

## 3.2.  安装完成：查看MPP目录结构
```bash
tree

/usr/local/
├── bin
│	├──  mpi_dec_test
│	├──  mpi_dec_mt_test
│	├──  mpi_dec_multi_test
│	├──  mpi_dec_nt_test
│	├──  mpi_enc_mt_test
│	├──  mpi_enc_test
│	├──  mpi_rc2_test
│	├──  mpp_info_test
│	├──  test_rknn_demo.sh
│	├──  test_rtsp.sh
│	└──  vpu_api_test
│
└── include/
│	└── rockchip
│			
└── lib
	├── librockchip_mpp.so
	├── librockchip_mpp.so.0
	├── librockchip_mpp.so.1
	├── librockchip_vpu.so
	├── librockchip_vpu.so.0
	├── librockchip_vpu.so.1
	└── pkgconfig

```



- 查看mpp对应的库文件：
	```bash
	ls /usr/local/lib
	
	librockchip_mpp.so    librockchip_mpp.so.1  librockchip_vpu.so.0  pkgconfig
	librockchip_mpp.so.0  librockchip_vpu.so    librockchip_vpu.so.1  
	```

- 查看mpp对应的头文件：

	```bash
	ls /usr/local/include/rockchip/

	mpp_buffer.h   mpp_log.h      mpp_task.h         rk_vdec_cfg.h  rk_venc_ref.h
	mpp_compat.h   mpp_meta.h     rk_hdr_meta_com.h  rk_vdec_cmd.h  vpu_api.h
	mpp_err.h      mpp_packet.h   rk_mpi_cmd.h       rk_venc_cfg.h  vpu.h
	mpp_frame.h    mpp_rc_api.h   rk_mpi.h           rk_venc_cmd.h
	mpp_log_def.h  mpp_rc_defs.h  rk_type.h          rk_venc_rc.h
	```

- 查看mpp对应的bin文件：	

	```bash
	ls /usr/local/bin
	
	mpi_dec_mt_test     mpi_dec_test     mpi_rc2_test       test_rtsp.sh
	mpi_dec_multi_test  mpi_enc_mt_test  mpp_info_test      vpu_api_test
	mpi_dec_nt_test     mpi_enc_test     test_rknn_demo.sh
	```
	
- 编解码器demo：
	
	

	```bash
	mpp_dec_test： 单线程解码器demo
	mpi_dec_mt_test：多线程解码器demo
	mpi_dec_multi_test：多实例解码器demo
		
	mpp_enc_test：单线程编码器demo
	mpi_enc_multi_test：多实例编码器demo
	```

- 实用工具
	MPP 提供了一些单元测试用的工具程序，这种程序可以对软硬件平台以及 MPP 库本身进行测试
	```bash
	mpp_info_test：    读取和打印 MPP 库的版本信息
	mpp_buffer_test：  测试内核的内存分配器是否正常。
	mpp_mem_test：     测试C库的内存分配器是否正常。
	mpp_runtime_test： 测试一些软硬件运行时环境是否正常。
	mpp_platform_test：读取和测试芯片平台信息是否正常。
	```
	


## 3.4. 我们可以运行mpp_dec_test来判断mpp安装情况：

```bash
mpp_dec_test
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dcbb193e487b44fcb1c4e6f56712915c.png)