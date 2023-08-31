# 1. 简介
- [专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 开发环境：ArmSoM-W3开发板、hdmi显示器、hdmi输入源

# 2. Audio配置
DTS添加声卡配置：

```
--- a/arch/arm64/boot/dts/rockchip/rk3588-evb1-lp4.dtsi
+++ b/arch/arm64/boot/dts/rockchip/rk3588-evb1-lp4.dtsi
@@ -47,6 +47,26 @@ play-pause-key {
};
};
+ hdmiin_dc: hdmiin-dc {
+ compatible = "rockchip,dummy-codec";
+ #sound-dai-cells = <0>;
+ };
+ hdmiin-sound {
+ compatible = "simple-audio-card";
+ simple-audio-card,format = "i2s";
+ simple-audio-card,name = "rockchip,hdmiin";
+ simple-audio-card,bitclock-master = <&dailink0_master>;
+ simple-audio-card,frame-master = <&dailink0_master>;
+ status = "okay";
+ simple-audio-card,cpu {
+ sound-dai = <&i2s7_8ch>;
+ };
+ dailink0_master: simple-audio-card,codec {
+ sound-dai = <&hdmiin_dc>;
+ };
+ };
+&i2s7_8ch {
+ status = "okay";
+};
```
这里有一个问题需要注意了，在写文档的时候是基于SDK版本号：

> rk3588_linux_release_v1.0.5_20221120.xml

在后续的更新里面hdmirx-audio使用的驱动变了

```
hdmiin-sound {
	    compatible = "rockchip,hdmi";
	    rockchip,mclk-fs = <128>;
	    rockchip,format = "i2s";
	    rockchip,bitclock-master = <&hdmirx_ctrler>;
	    rockchip,frame-master = <&hdmirx_ctrler>;
	    rockchip,card-name = "rockchip,hdmiin";
	    rockchip,cpu = <&i2s7_8ch>;
	    rockchip,codec = <&hdmirx_ctrler 0>;
	    rockchip,jack-det;
	};
```
这里只需了解就行了

# 3.声卡信息
开发板上电时会有如下打印：

```
[    6.640779] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_delayed_work_audio: enable audio
[    6.640801] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_delayed_work_audio: restart audio fs(44100 -> 44100) ch(0 -> 2)
[    6.640810] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_audio_fifo_init
[    6.847421] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_delayed_work_audio: audio underflow 0x2000000, with fs valid 44100
[    6.847443] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_audio_fifo_init
[    7.257421] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_delayed_work_audio: restart audio fs(44100 -> 48000) ch(2 -> 2)
[    7.257468] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_audio_fifo_init
[    7.460747] rk_hdmirx fdee0000.hdmirx-controller: audio on
[   24.984083] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_delayed_work_audio: audio underflow 0x2000000, with fs valid 44100
[   24.984117] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_audio_fifo_init
[   33.727417] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_delayed_work_audio: audio underflow 0x2000000, with fs valid 44100
[   33.727456] rk_hdmirx fdee0000.hdmirx-controller: hdmirx_audio_fifo_init
```
这里包含hdmirx-audio采样率和通道数的信息
当然你也可以在驱动文件里面/sys/class/hdmirx/hdmirx查看信息

使用如下命令可以查看声卡信息：

```
cat /proc/asound/cards
 0 [rockchiphdmi0  ]: rockchip-hdmi0 - rockchip-hdmi0
                      rockchip-hdmi0
 1 [rockchiphdmi1  ]: rockchip-hdmi1 - rockchip-hdmi1
                      rockchip-hdmi1
 2 [rockchiphdmiin ]: rockchip_hdmiin - rockchip,hdmiin
                      rockchip,hdmiin
 3 [rockchipes8311 ]: rockchip_es8311 - rockchip,es8311
                      rockchip,es8311
```

# 4.声音测试
在Linux系统里面默认是以arecord和aplay命令来测试声音

```
录音：
root@linaro-alip:~# arecord -l
**** List of CAPTURE Hardware Devices ****
card 2: rockchiphdmiin [rockchip,hdmiin], device 0: rockchip,hdmiin i2s-hifi-0 [rockchip,hdmiin i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 3: rockchipes8311 [rockchip,es8311], device 0: fe470000.i2s-dummy_codec fpc_dc-0 [fe470000.i2s-dummy_codec fpc_dc-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
  
  //arecord可以查看当前系统里有哪些声卡可以录音，以上内容可以看到hdmirx是card2
  
 root@linaro-alip:~# arecord -D hw:2,0 -f S16_LE -r 48000 -c 2 -d 2 t.wav
Recording WAVE 't.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Stereo
//声音格式：16位 采样率：48000 通道数：2 录音：2
```
使用上述命令录音可以得到t.wav文件，之后使用aplay测试声音

```
root@linaro-alip:~# aplay -D plughw:1,0 t.wav  //指定hdmiout播放
Playing WAVE 't.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Stereo
```

# 5.经验分享

上述命令在测试声卡阶段没有问题，但是作为开发者来这远不足够，大部分会使用GStreamer对音频进行处理，接下来分享一下我使用GStreamer对hdmirx处理的时候遇到的一些问题
音频有两个问题：

> 1，有延时情况下，无法使用leaky=1，导致我们视频和音频的同步出现了严重问题。 
> 2，通过hdmi播放音频有（哒哒哒）杂音 命令如下：
> 1，无延时：gst-launch-1.0 alsasrc device=hw:2,0 ! queue ! volume mute=false
> ! alsasink device=hw:1,0 -e -v
>  2，有延时：gst-launch-1.0 alsasrc
> device=hw:2,0 use-driver-timestamps=false ! queue  max-size-buffers=0
> max-size-bytes=0 max-size-time=3100000000 min-threshold-buffers=0
> min-threshold-bytes=0 min-threshold-time=3000000000 ! volume
> mute=false ! alsasink device=hw:1,0 async=true -e -v

gst-launch-1.0 alsasrc device=hw:2,0 use-driver-timestamps=false ! queue leaky=1  max-size-buffers=0 max-size-bytes=0 max-size-time=3100000000 min-threshold-buffers=0 min-threshold-bytes=0 min-threshold-time=3000000000 ! volume mute=false ! alsasink device=hw:1,0 async=true -e -v
解决方法：
更新/usr/lib/aarch64-linux-gnu/gstreamer-1.0/libgstcoreelements.so

通过hdmi播放音频有（哒哒哒）杂音 ：
分析是根据采样率导致的，gst-launch-1.0 alsasrc device=hw:2,0 ! queue ! volume mute=false ! alsasink device=hw:1,0 -e -v会使用没指定采样率，会使用默认的，
采样率多少可以查看/sys/class/hdmirx/hdmirx/audio_rate
正确的命令应该是：

> gst-launch-1.0 alsasrc device=hw:2,0 ! audio/x-raw,format=S16LE,rate=48000,chanels=8 ! queue ! volume mute=false ! alsasink device=hw:1,0 -e -v

而采样率多少是根据音频源决定的
![在这里插入图片描述](https://img-blog.csdnimg.cn/79b2faf55de442a38453cfe2f8ad13a9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d3aa79aa9bba47a183361aaa93ab9e82.png)
gstreamer之前有杂音是因为自动协商出来了错的rate，默认的采样率是44100Hz

## 6.结语
开发者对于音频来说需要给到更多参数：采样率、码率、通道数 、state，
能在驱动文件里面得到这些信息有利于更好做开发，对声音有最直接影响的是采样率。其他参数，码率=采样率 x 声道数 x 位深也是是可以算出来，其他参数的影响不大。

## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)