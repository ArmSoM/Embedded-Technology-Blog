ArmSoM正计划推出一款新的产品，这款产品将采用强大的RK3576芯片。

本文将为您介绍我们的新产品搭载的RK3576性能参数，以及它如何为您提供卓越的性能和功能。

## RK3576处理器

RK3576处理器是一款强大的处理器，具备出色的性能和多样化的功能，非常适合EINK产品，扫地机器人，云端产品，用于嵌入式系统和智能设备。它采用先进的制程工艺，拥有高性能的多核CPU和强大的图形处理单元，能够处理各种复杂任务，如图像处理、视频播放和音频处理。
![RK3567-Rockchip SoC Roadmap for AIOT](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/rk3576/RK3567-Rockchip-SoC-Roadmap-for-AIOT.png)
以下为RK3576芯片规格
* CPU：Quad A72 + Quad A53 CPU
* GPU：G52 MC3 @ 1GHz
* DDR: 32-bit LPDDR4/LPDDR4x/LPDDR5
* 编解码：
    * 解码支持4K@120fps H.265/H.264/AV1/VP9/AVS2
    * 编码支持4K@30fps H.264/H.265
    * 编解码支持4K@30fps MJPG
* 显示：
    * 支持 2.5K+2.5K+2K or 4K+2K
    * HDMI 2.1/eDP 1.3 Combo TX @ 4K60
    * MIPI DSI-2 TX @ 4K60
    * DisplayPort 1.4 TX @ 4K60
    * EPD (Electronic Paper Display) TX @ 2560x1920
    * Parallel interface (RGB888) TX @ 1080P60
* 视频输入
    * 16M Pixel ISP with HDR (up to 120dB)
    * 一个 MIPI CSI-2 with D-PHY (4x1, 2x2)
    * 一个 MIPI CSI-2 with C/D-PHY (4x1)
    * 一个 DVP 8/10/12/16-bit
* 高速接口
    * 一个 USB 3.2 Gen1 supports type-C AltMode with DP
    * 一个 PCIe 2.1/SATA 3.1/USB 3.2 Gen1 combo port
    * 一个 PCIe 2.1/SATA 3.1 combo port
    * 双 RGMII 接口

* 其他
    * CAN, I3C, I2C, SPI, UART, GPIO
    * I2S/TDM/PCM (2x 4T4R, 3x 2T2R), 2x 8CH PDM, 2x S/PDIF
    * ASRC (2x 2CH + 2x 4CH)
* 安全
    * ARM TrustZone security extension
    * Secure boot / key ladder / OTP
    * Cipher engine (RSA, ECC, HASH, DES, AES, SHA, SM)

![RK3576-series product comparison chart](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/rk3576/RK3576-series-product-comparison-chart.png)
## 应用场景

### EINK产品应用

**更强的显示能力**

* 支持EPD显示，分辨率可以支持到1920*2560
* RKPQ，增强清晰度等，给用户带来更好的阅读体验
* 多种刷新模式:全局、局部、快速、极速、连续
* 对比度调节，抖动算法，增彩，增亮，灰度过滤等图像处理算法
* 支持DP(TypeC)显示扩展，连接显示器进行办公

**更高的翻页和书写性能**

* 8核CPU，且有A72大核心，高频率等带来高计算能力
* 支持UFS 2.1，超高数据读取速度(12Gbps)
* LP4/4X-4266和LP5-4800，更高速率，更大带宽
* 支持PCleWiFi，更快的网络存取速度
* RK自研刷新算法，手写快速流畅(20ms延时)，支持彩色手写

**更低的功耗，更长的续航**

* 先进制程，保证高性能的同时带来更低功耗
* 低功耗待机模式(3.8V @0.6mA左右@LP4X)

**更强的智能化扩展**

* 支持6TOPSNPU，更多算子
* 更好的语音转文字、手写转文字体验、OCR识别

### 扫地机器人产品

**更强的计算能力**
* 四核A72+四核A53，频率高至2.2GHZ，计算性能强劲
* NEON协处理器可用于SIMD类计算
* 145GFLOPS的GPU可以支持有效的异构计算、鱼眼矫正
* 支持6TOPSNPU，更多算子，双核架构支持并行计算

**更强的图像处理能力**
* 16MP ISP，支持低光噪
* 支持RGB-IR sensor
* 支持最高120dB HDR
* AI-ISP提升低噪度的图像效果

**更低的功耗，更长的续航**
* 先进制程，保证高性能的同时带来更低功耗
* 低功耗待机模式(3.8V @0.5mA左右)

**更高的扩展能力**
* 支持PCle接口WiFi，图传更快速

### 云端产品应用（云盒子/云一体机/云笔电）

**更强的显示能力**
* 支持H265YUV422/YUV444解码能力，满足无损类产品应用
* 支持4K显示、4KUl，HDMI/DP/MIPI等多种显示接口，最多支持三屏显示
* RK PQ，多种增强画质算法，给用户带来更好的阅读体验

**更高的产品性能**
* 8核CPU，且有A72大核心，高频率等带来高计算能力
* 支持UFSv2.1，超高数据读取速度(12Gbps)
* LP4/4X-4266和LP5-4800，高速率，更大带宽
* 支持PCleWiFi，更快的网络存取速度
* 支持TypeC、SATA、USB3等高速接口

**更低的功耗，更长的续航**
* 先进制程，保证高性能的同时带来更低功耗
* 低功耗待机模式(3.8V @0.5mA左右)，有利于笔电类产品

**丰富的方案经验**
* 满足不同产品需求成熟的Linux、Android SDK

## ArmSoM 产品规划
基于以上优势，ArmSoM将研发一款基于RK3576的开发套件，产品框图后续将公布，请持续关注armsom官方账号。
