下面是Jetson Nano模块（以NVIDIA Jetson Nano为代表）与armsom开发的AIM5（Rockchip RK3588）的主要技术规格的对比，整理成表格：

| 规格                                 | Jetson Nano (NVIDIA)                | ArmSoM-AIM5              |
|--------------------------------------|-----------------------------------|--------------------------------|
| CPU核数    | 四核 ARM® Cortex®-A57 MPCore 处理器    | 四核ARM® Cortex®-A76+四核 ARM® Cortex®-A55 |
| GPU核数    | 128核Maxwell架构GPU   | ARM Mali-G610 MP4                 |
| 内存容量   | 4GB 64位 LPDDR4, 1600MHz   | 8GB/32GB 64位 LPDDR4x, 2112Mhz  |
| 存储支持  | microSD卡、16GB eMMC 5.1闪存  | microSD卡、32GB eMMC 5.1 闪存   |
| 视频编码  | 250 MP/sec，1x 4K@30 (HEVC)，2x 1080p@60 (HEVC)，4x 1080p@30 (HEVC)  |  8K@30fps H.265 / H.264  |
| 视频解码  | 500 MP/s，1x 4K@60 (HEVC)，2x 4K@30 (HEVC)，4x 1080p@60 (HEVC)，8x 1080p@30 (HEVC)  |  8K@60fps H.265/VP9/AVS2，8K@30fps H.264 AVC/MVC，4K@60fps AV1，1080P@60fps MPEG-2/-1/VC-1/VP8 |
| USB端口  | 1 个 USB 3.0、3 个 USB 2.0  | 1 个 USB 3.0、3 个 USB 2.0  |
| 以太网接口    | 1 个 10/100/1000 BASE-T 以太网 | 1 个 10/100/1000 BASE-T 以太网   |
| CSI接口 | 12 通道（3x4 或 4x2）MIPI CSI-2 D-PHY 1.1 (18 Gbps)     | 12通道(4x2)MIPI CSI-2 D-PHY1.1(18 Gbps)      |
| I/O        | 3 个 UART、2 个 SPI、2 个 I2S、4 个 I2C、多个 GPIO        | 3 个 UART、2 个 SPI、2 个 I2S、4 个 I2C、多个 GPIO   |
| PCIE    | 1 个 1/2/4lan PCIE2.0  | 1 个 1/2/4lan PCIE3.0 & 1 个 1lan PCIE2.0   |
| HDMI输出      | 1 个 HDMI 2.0  | 1 个 HDMI OUT2.1  / 1 个 eDP 1.4   |
| DP接口    | 1 个 DP1.2  | 1 个 DP1.4a |
| eDP/DP接口    | 1 个 eDP 1.4/1 个 DP接口  | 1 个 eDP 1.4/ 1 个 HDMI OUT2.1  |
|  DSI接口    | 1 个 DSI (1 x2) 2 同步  | 1 个 DSI (1 x2) 2 同步 |
| 操作系统支持    | NVIDIA JetPack软件套件 | 支持debian，ubuntu，armbian，kernel版本都为5.10                 |
| 大小   | 69.6 mm x 45 mm  | 69.6 mm x 45 mm |
|规格尺寸|260 引脚边缘连接器| 260 引脚边缘连接器|

这个表格对Jetson Nano模块和ArmSoM-AIM5技术规格进行了对比，以帮助您更容易了解它们的不同特性。

### ArmSoM-AIM5 对比 Jetson Nano的优势: 100%兼容且更优

![JetsonNano_product](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/Jetson_nano/JetsonNano_product.png)

**优势一：更强的视频编解码能力**

armsom-AIM5 采用瑞芯微最新的第四代编解码技术，真8K视频图像更清晰、细节更丰富。根据产品需求，还可以支持更高分辨率比如12K视频的播放，除了通用的H265/H264之外，8K VP9/AV1解码的支持，可以很好兼容海外视频内容的播放。AI感知编码、多ROI、低延迟编码、智能码率控制等编码技术的升级，给特定场景比如安防等应用提供了更优化的性能。

**优势二：更丰富的接口**

保持和Jeston Nano 引脚定义相同时，我们在Jeston Nano预留的引脚上加了一组 pcie2.0。Jeston Nano的pcie2.0接口，AIM5提供pcie3.0，为客户提供更丰富的接口选择。

**优势三：拥有更大的内存空间**

armsom-AIM5 可以支持32GB内存，更大的内存意味着更多的可能。
1. **更好的多媒体体验**：视频编辑、游戏和图形设计等多媒体任务通常需要大内存来处理和编辑高分辨率媒体文件，带来更好的多媒体体验。
2. **更快的应用程序响应时间**：内存中存储的数据可以更快地访问，从而提高应用程序的响应时间。这意味着应用程序会更加流畅和高效。
3. **数据缓存**：内存中的数据可以用作缓存，以加速数据访问。这对于数据库、Web服务器和其他数据密集型应用程序非常有用。
4. **未来扩展性**： 随着应用程序和操作系统变得越来越复杂，更大的内存容量可以使设备在未来保持竞争力，不必频繁升级硬件。

更大的内存空间有助于提高计算机和数字设备的性能、响应速度和多任务处理能力，armsom-AIM5 能够更有效地应对各种计算需求。
