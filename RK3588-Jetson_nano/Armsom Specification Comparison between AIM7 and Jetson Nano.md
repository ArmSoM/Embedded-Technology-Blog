# Armsom: Specification Comparison between AIM5 and Jetson Nano

The following table compares the key specifications between the Jetson Nano module represented by NVIDIA product and the AIM5 developed by ArmSoM based on Rockchip RK3588:

| Specifications   | Jetson Nano (NVIDIA)                                         | ArmSoM-AIM7(Rockchip)                                       |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| CPU Cores        | Quad-core ARM® Cortex®-A57 MPCore processor                  | Quad-core ARM® Cortex®-A76 + Quad-core ARM® Cortex®-A55      |
| GPU Cores        | 128-core Maxwell GPU                                         | ARM Mali-G610 MP4                                            |
| Memory           | 4GB 64-bit LPDDR4, 1600MHz                                   | 8GB/32GB 64-bit LPDDR4x, 2112MHz                             |
| Storage          | microSD card, 16GB eMMC 5.1 flash storage                    | microSD card, 32GB eMMC 5.1 flash storage                    |
| Video Encoding   | 250 MP/sec, 1x 4K@30 (HEVC), 2x 1080p@60 (HEVC), 4x 1080p@30 (HEVC) | 8K@30fps H.265 / H.264                                       |
| Video Decoding   | 500 MP/s, 1x 4K@60 (HEVC), 2x 4K@30 (HEVC), 4x 1080p@60 (HEVC), 8x 1080p@30 (HEVC) | 8K@60fps H.265/VP9/AVS2, 8K@30fps H.264 AVC/MVC, 4K@60fps AV1, 1080P@60fps MPEG-2/-1/VC-1/VP8 |
| USB Ports        | 1 USB 3.0, 3 USB 2.0                                         | 1 USB 3.0,  3 USB 2.0                                        |
| Ethernet         | 1 10/100/1000 BASE-T                                         | 1 10/100/1000 BASE-T                                         |
| CSI Interfaces   | 12 channels (3x4 or 4x2) MIPI CSI-2 D-PHY 1.1 (18 Gbps)      | 12 channels (4x2) MIPI CSI-2 D-PHY1.1 (18 Gbps)              |
| I/O              | 3 UARTs, 2SPIs, 2 I2S, 4 I2Cs, multiple GPIOs                | 3 UARTs, 2 SPIs, 2 I2S, 4 I2Cs, multiple GPIOs               |
| PCIE             | 1 1/2/4lane PCIE2.0                                          | 1 1/2/4lane PCIE3.0 & 1 1lane PCIE2.0                        |
| HDMI Output      | 1 HDMI 2.0                                                   | 1 HDMI OUT2.1 / 1 eDP 1.4                                    |
| DP Interface     | 1 DP1.2                                                      | 1 DP1.4a                                                     |
| eDP/DP Interface | 1 eDP 1.4 / 1 DP                                             | 1 eDP 1.4 / 1 HDMI OUT2.1                                    |
| DSI Interface    | 1 DSI (1 x2) 2 sync                                          | 1 DSI (1 x2) 2 sync                                          |
| OS Support       | NVIDIA JetPack software suite                                | Support debian, ubuntu, armbian, kernel 5.10                 |
| Size             | 69.6 mm x 45 mm                                              | 69.6 mm x 45 mm                                              |
| Form Factor      | 260-pin edge connector                                       | 260-pin edge connector                                       |

This table compares the specifications of Jetson Nano module and ArmSoM-AIM5 to help understand their different features. 

### Advantages of ArmSoM-AIM5 over Jetson Nano: 100% compatible and better

**Strength 1: Stronger video encoding and decoding capabilities**

ArmSoM-AIM5 adopts Rockchip's latest 4th generation codec technology for clearer 8K video images and richer details. It can support higher resolutions like 12K video playback based on product requirements.In addition to universal H.265/H.264 support, 8K VP9/AV1 decoding support can better accommodate overseas video content playback. 

Upgrades to encoding technologies like AI-aware encoding, multi-ROI, low latency encoding, intelligent bitrate control provide more optimized performance for scenarios like security.

**Strength 2: More abundant interfaces** 

While maintaining pin compatibility with Jetson Nano, ArmSoM added a set of PCIE 2.0 to the reserved pins of Jetson Nano.

 AIM5 provides PCIE 3.0 compared with Jetson Nano's PCIE 2.0 interface, providing customers more interface options.

**Strength 3: Larger memory space**

ArmSoM-AIM5 can support up to 32GB memory, meaning more possibilities:

1. Better multimedia experience for video editing, gaming, graphics design which require large memory to process high resolution media files.

2. Faster application response times as data in memory can be accessed faster.Running application will be more smoothly and efficiently.

3. Data caching in memory can accelerate data access, very useful for databases, web servers, etc. 

4. Future expandability as larger memory helps devices stay competitive without frequent hardware upgrades as software complexity increases.

The increased memory space of ArmSoM-AIM5 helps improve performance, response speed, and multitasking capabilities to handle diverse computing needs more effectively.