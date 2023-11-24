# RK3588 Encoding and Decoding Analysis of Mpp Encoding Demo：mpi_enc_test

# I. Introduction 

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)
- mpi_enc_test is the official encoding demo from Rockchip
- This article analyzes the code and encoding flow of mpi_enc_test

# II. Environment Introduction

- Hardware environment:  
  ArmSoM-W3 RK3588 Development Board

- Software version: 
  OS: ArmSoM-W3 Debian11

# III. Mpp Encode/Decode Flow Analysis 

![image](https://img-blog.csdnimg.cn/27dfa966953446019431ca9f7e5b7e62.png#pic_center)

<center>Fig. 3.1 RKMPP encoder interface provides functions for inputting image data and outputting stream  

- mpp_create: get MppCtx instance and MppApi structure
- mpp_init: init encoding/decoding type and format of MppCtx
- mpi->control: configure encoding/decoding parameters through commands 
- encode_put_frame: input image data MppFrame, e.g. YUV, RGB
- encode_get_packet: get encoded stream data stored in MppPacket, e.g. H.264, H.265  

- mpi->reset: reset decoder to normal initialized status
- mpp_destroy: release allocated memory, destroy properly  

# IV. Important Function Analysis  

## 4.1 mpp_init function: init encoding/decoding type and format of MppCtx

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

- MppCtxType parameter: init decoder or encoder

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
  
  Refer to rk_type.h for more definitions
  
  
## 4.2 Set encoder parameters: test_mpp_enc_cfg_setup  

Function prototype:   

```cpp
MPP_RET test_mpp_enc_cfg_setup(MpiEncMultiCtxInfo *info) 
```

Function analysis:  
Mainly sets encoder parameters and saves encoder parameters to MpiEncMultiCtxInfo structure

## 4.3 Encode function: test_mpp_run   

Function prototype:   

```cpp
MPP_RET test_mpp_run(MpiEncMultiCtxInfo *info)
```

Function analysis:  
Input image data MppFrame,and get encoded stream data stored in MppPacket  


# V. mpi_enc_test Flow Analysis   

**mpi_enc_test encode command example:**   

```bash  
sudo mpi_enc_test -i /oem/decode.yuv -t 7 -n 200 -o /oem/encode.h264 -w 1920 -h 1080 -fps 60
```

**mpi_enc_test flow analysis:**   

```bash
main -> enc_test_multi -> pthread_create(enc_test) -> enc_test -> test_mpp_enc_cfg_setup -> test_mpp_run -> reset -> mpp_destroy -> pthread_join -> MPP_FREE  
```

Main flow: init ---> test_mpp_enc_cfg_setup set related encoder params --- > test_mpp_run(info): input image data, output stream  

- The main function parses parameters based on input cmd params <font color="red" size="3">(cmd param char **argv corresponds to command -i /oem/decode.yuv -t 7 -n 200 -o /oem/encode.h264 -w 1920 -h 1080 -fps 60)</font> and saves to structure <font color="red" size="3">MpiEncTestArgs *cmd</font>   
- <font color="red" size="3">enc_test_multi(cmd, argv[0]);</font> It is the encapsulated encode function. Pass in structure <font color="red" size="3">MpiDecTestCmd * cmd</font> to achieve H.265/H.264 video encoding with concurrent multi-channel processing  
- <font color="red" size="3">enc_test_multi(cmd, argv[0]);</font> uses for loop to create new threads and call enc_test to execute encoding tasks
- Function <font color="red" size="3">enc_test</font> is the entire encode flow function. First executes some MPP init operations: mpp_create() mpp_init(), mpp_enc_cfg_init(), mpi->control. After init, execute <font color="red" size="3">test_mpp_enc_cfg_setup</font> function to set related encoder parameters. Then the most important encoding operation: <font color="red" size="3">test_mpp_run(info)</font>: input image data, and output stream   
- After encoding, execute <font color="red" size="3">reset</font> : <font color="red" size="3">p->mpi->reset(p->ctx)</font> reset encoder to normal initialized status  
- After encoder reset, <font color="red" size="3">release allocated memory through mpp_destroy()</font>, some destroy operations to prevent memory leak
- Function <font color="red" size="3">pthread_join</font> waits for encode threads to end, then calls <font color="red" size="3">MPP_FREE(ctxs)</font> to release threads  
- Encode threads end, goes back to main() function of main thread, and executes <font color="red" size="3">mpi_enc_test_cmd_put(cmd),</font> to release memory occupied by cmd object   

# VI. mpi_enc_test Usage Example   

- Execute encode command in terminal:   

  ```bash
  sudo mpi_enc_test -i /oem/decode.yuv -t 7 -n 200 -o /oem/encode.h264 -w 1920 -h 1080 -fps 60 
  ```
  
- Convert decode.yuv to h264, resolution 1920x1080, fps60, total 200 frames

- Analysis of command： -i ：input file, -t 7： output H.264 stream, -n 200 ：encode 200 frames, -w ：image width, -h： image height, fps 60  

- View encode/decode output information:   

  ```bash 
  tail -f /var/log/syslog    //open new terminal, monitor output
   
  sudo mpi_enc_test -i /oem/decode.yuv -t 7 -n 200 -o /oem/encode.h264 -w 1920 -h 1080 -fps 60   //Turn to original terminal, run encode program
  ```

  Encode output as below: 


```bash
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: cmd parse result:
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: input  file name: /oem/decode.yuv
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: output file name: /oem/encode.h264
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: width      : 1920
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: height     : 1080
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: format     : 0
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_utils: type       : 7
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: mpi_enc_test start
Jun 18 16:20:47 linaro-alip mpp[2139]: mpp_info: mpp version: 8a54ab8d author: xueman.ruan   2023-06-16 [h264d]: fix the derivation of mbaff.
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: 0x7fa0001960 encoder test start w 1920 h 1080 type 7
Jun 18 16:20:47 linaro-alip mpp[2139]: mpp_enc: MPP_ENC_SET_RC_CFG bps 15552000 [972000 : 16524000] fps [60:60] gop 120
Jun 18 16:20:47 linaro-alip mpp[2139]: h264e_api_v2: MPP_ENC_SET_PREP_CFG w:h [1920:1080] stride [1920:1080]
Jun 18 16:20:47 linaro-alip mpp[2139]: mpp_enc: mode vbr bps [972000:15552000:16524000] fps fix [60/1] -> fix [60/1] gop i [120] v [0]
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 0    size 28319   qp 10
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 1    size 32698   qp 14
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 2    size 45308   qp 15
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 3    size 40435   qp 15
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 4    size 45222   qp 15
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 5    size 36992   qp 17
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 6    size 44143   qp 17
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 7    size 30927   qp 17
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 8    size 40750   qp 16
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 9    size 49589   qp 18
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 10   size 38120   qp 18
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 11   size 31903   qp 17
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 12   size 44712   qp 17
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 13   size 43897   qp 19
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 14   size 45241   qp 19
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 15   size 36346   qp 19
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 16   size 29973   qp 18
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 17   size 37295   qp 20
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 18   size 51772   qp 20
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 19   size 33378   qp 20
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 20   size 33600   qp 19
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 21   size 37763   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 22   size 49116   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 23   size 31925   qp 20
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 24   size 46841   qp 19
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 25   size 36214   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 26   size 46354   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 27   size 37565   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 28   size 46092   qp 20
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 loop times 1
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 29   size 15581   qp 20
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 30   size 23445   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 31   size 32401   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 32   size 28462   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 33   size 33358   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 34   size 28697   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 35   size 35228   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 36   size 24997   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 37   size 31774   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 38   size 39806   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 39   size 31097   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 40   size 25497   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 41   size 36976   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 42   size 36927   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 43   size 37975   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 44   size 31915   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 45   size 26709   qp 21
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 46   size 33195   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 47   size 46155   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 48   size 29497   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 49   size 29307   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 50   size 33197   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 51   size 42712   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 52   size 28279   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 53   size 40612   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 54   size 33483   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 55   size 40900   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 56   size 32802   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 57   size 39739   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 loop times 2
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 58   size 14135   qp 22
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 59   size 21344   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 60   size 29755   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 61   size 25979   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 62   size 30094   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 63   size 26150   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 64   size 32448   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 65   size 22886   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 66   size 28925   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 67   size 36458   qp 25
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 68   size 28537   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 69   size 23470   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 70   size 33644   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 71   size 33987   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 72   size 34409   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 73   size 28965   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 74   size 23782   qp 23
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 75   size 30565   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 76   size 42286   qp 25
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 77   size 26486   qp 25
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 78   size 26157   qp 24
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 79   size 30454   qp 25
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 80   size 39307   qp 26
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 81   size 26085   qp 25
Jun 18 16:20:47 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 82   size 36866   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 83   size 31373   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 84   size 38846   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 85   size 31552   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 86   size 37866   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 loop times 3
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 87   size 13688   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 88   size 20366   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 89   size 28290   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 90   size 24927   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 91   size 28663   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 92   size 25063   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 93   size 30803   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 94   size 21575   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 95   size 27338   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 96   size 34735   qp 26
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 97   size 27243   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 98   size 22440   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 99   size 33794   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 100  size 33975   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 101  size 34414   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 102  size 28039   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 103  size 23884   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 104  size 30492   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 105  size 42340   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 106  size 26504   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 107  size 26186   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 108  size 32429   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 109  size 41124   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 110  size 25814   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 111  size 36782   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 112  size 31428   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 113  size 38736   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 114  size 31537   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 115  size 37914   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 loop times 4
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 116  size 13688   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 117  size 20324   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 118  size 28272   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 119  size 24945   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 120  size 50373   qp 15
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 121  size 25603   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 122  size 32347   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 123  size 22828   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 124  size 28898   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 125  size 36563   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 126  size 28604   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 127  size 23537   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 128  size 33637   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 129  size 34220   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 130  size 34337   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 131  size 29011   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 132  size 23784   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 133  size 30583   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 134  size 42288   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 135  size 28161   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 136  size 27757   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 137  size 32119   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 138  size 41349   qp 25
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 139  size 26902   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 140  size 38575   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 141  size 32464   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 142  size 40765   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 143  size 32726   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 144  size 39886   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 loop times 5
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 145  size 14182   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 146  size 21394   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 147  size 29792   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 148  size 27333   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 149  size 31698   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 150  size 27641   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 151  size 33985   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 152  size 23571   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 153  size 30157   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 154  size 37937   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 155  size 29964   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 156  size 24665   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 157  size 35541   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 158  size 35537   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 159  size 35918   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 160  size 30675   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 161  size 24880   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 162  size 31974   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 163  size 43998   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 164  size 29454   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 165  size 29434   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 166  size 33309   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 167  size 42905   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 168  size 28273   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 169  size 40610   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 170  size 33655   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 171  size 42553   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 172  size 32763   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 173  size 39818   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 loop times 6
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 174  size 14135   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 175  size 22401   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 176  size 31085   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 177  size 27330   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 178  size 31714   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 179  size 27454   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 180  size 33911   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 181  size 23618   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 182  size 30246   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 183  size 38036   qp 24
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 184  size 29962   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 185  size 24672   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 186  size 35546   qp 22
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 187  size 35542   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 188  size 35918   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 189  size 30675   qp 23
Jun 18 16:20:48 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 190  size 24880   qp 22
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 191  size 31974   qp 23
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 192  size 43998   qp 24
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 193  size 29454   qp 23
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 194  size 29434   qp 22
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 195  size 33309   qp 23
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 196  size 42905   qp 24
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 197  size 28273   qp 23
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 198  size 40610   qp 22
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encoded frame 199  size 33655   qp 23
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: chn 0 encode 200 frames time 1841 ms delay   4 ms fps 108.60 bps 15541070
Jun 18 16:20:49 linaro-alip mpp[2139]: mpi_enc_test: mpi_enc_test average frame rate 108.60



```