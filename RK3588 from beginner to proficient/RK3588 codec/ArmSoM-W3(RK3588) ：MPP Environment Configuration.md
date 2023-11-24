# ArmSoM-W3(RK3588): MPP Environment Configuration

# 1. Introduction

The Media Process Platform (MPP) provided by Rockchip is a general media processing software platform suitable for Rockchip chip series. 

The platform shields the complex underlying processing related to chips for application software. Its purpose is to shield the differences between different chips and provide application developers with a unified video media processing interface (Media Process Interface, abbreviated as MPI). 

The functions provided by MPP include:

- Video decoding
H.265/H.264/H.263/VP9/VP8/MPEG-4/MPEG-2/MPEG-1/VC1/MJPEG

- Video encoding 
H.264/VP8/MJPEG

- Video processing

- Video copy, Scaling, Color space conversion, Deinterlace

# 2. Environment Introduction

- Hardware environment: 
ArmSoM-W3 RK3588 development board

- Software version:
OS: ArmSoM-W3 Debian11

# 3. RK3588 MPP Environment Configuration 

## 3.1. Download and install rkmpp
- Download mpp package from github:

	```bash
	git clone https://github.com/rockchip-linux/mpp.git
	```

- Compile and install

	```bash
	cd mpp/build/linux/aarch64
	 
	./make-Makefiles.bash
	
	make -j8
	
	sudo make install     
	```

## 3.2. Installation completed: View MPP directory structure
```bash   
tree 

/usr/local/
├── bin
│   ├── mpi_dec_test
│   ├── mpi_dec_mt_test 
│   ├── mpi_dec_multi_test
│   ├── mpi_dec_nt_test
│   ├── mpi_enc_mt_test
│   ├── mpi_enc_test
│   ├── mpi_rc2_test
│   ├── mpp_info_test
│   ├── test_rknn_demo.sh  
│   ├── test_rtsp.sh
│   └── vpu_api_test
│   
└── include/
│   └── rockchip
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



- View mpp corresponding library files:
	```bash 
	ls /usr/local/lib
	
	librockchip_mpp.so    librockchip_mpp.so.1  librockchip_vpu.so.0  pkgconfig
	librockchip_mpp.so.0  librockchip_vpu.so    librockchip_vpu.so.1
	```

- View mpp corresponding header files:

	```bash
	ls /usr/local/include/rockchip/

	mpp_buffer.h   mpp_log.h      mpp_task.h         rk_vdec_cfg.h  rk_venc_ref.h
	mpp_compat.h   mpp_meta.h     rk_hdr_meta_com.h  rk_vdec_cmd.h  vpu_api.h
	mpp_err.h      mpp_packet.h   rk_mpi_cmd.h       rk_venc_cfg.h  vpu.h 
	mpp_frame.h    mpp_rc_api.h   rk_mpi.h           rk_venc_cmd.h
	mpp_log_def.h  mpp_rc_defs.h  rk_type.h          rk_venc_rc.h
	```

- View mpp corresponding bin files:	

	```bash
	ls /usr/local/bin
	
	mpi_dec_mt_test     mpi_dec_test     mpi_rc2_test       test_rtsp.sh
	mpi_dec_multi_test  mpi_enc_mt_test  mpp_info_test      vpu_api_test
	mpi_dec_nt_test     mpi_enc_test     test_rknn_demo.sh
	```
	
- Encoder/decoder demos:
	

	```bash 
	mpp_dec_test: single thread decoder demo
	mpi_dec_mt_test: multi-thread decoder demo
	mpi_dec_multi_test: multi-instance decoder demo
		
	mpp_enc_test: single thread encoder demo
	mpi_enc_multi_test: multi-instance encoder demo
	```

- Utility tools
	MPP provides some unit test tools that can test the software/hardware platform and the MPP library itself:
	
	```bash
	mpp_info_test: read and print MPP library version info
	mpp_buffer_test: test kernel memory allocator 
	mpp_mem_test: test C library memory allocator
	mpp_runtime_test: test some software and hardware on runtime environment 
	mpp_platform_test: read and test chip platform info
	```
	

## 3.4. Run mpp_dec_test to check mpp installation:

```bash 
mpp_dec_test
```

![insert image description here](https://img-blog.csdnimg.cn/dcbb193e487b44fcb1c4e6f56712915c.png)