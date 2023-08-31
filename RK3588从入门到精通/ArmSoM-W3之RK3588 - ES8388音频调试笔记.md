# 1. 简介
- [专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)
- 本文是基于RK3588平台，音频芯片ES8388调试总结。
- 外接声卡：ES8388
# 2. 音频ES8388调试
## 2.1 调试总览，调试步骤分析
- 步骤 ①  dts配置
- 步骤 ②  编译烧写，调试

 ## 2.2 dts配置



- **系统声音配置：**
	```bash
	es8388_sound: es8388-sound {
	        status = "okay";
	        compatible = "rockchip,multicodecs-card";
	        rockchip,card-name = "rockchip-es8388";
	        hp-det-gpio = <&gpio1 RK_PD5 GPIO_ACTIVE_LOW>;
	        io-channels = <&saradc 3>;
	        io-channel-names = "adc-detect";
	        keyup-threshold-microvolt = <1800000>;
	        poll-interval = <100>;
	        spk-con-gpio = <&gpio1 RK_PD3 GPIO_ACTIVE_HIGH>;
	        hp-con-gpio = <&gpio1 RK_PD2 GPIO_ACTIVE_HIGH>;
	        rockchip,format = "i2s";
	        rockchip,mclk-fs = <256>;
	        rockchip,cpu = <&i2s0_8ch>;
	        rockchip,codec = <&es8388>;
	        rockchip,audio-routing =
	            "Headphone", "LOUT1",
	            "Headphone", "ROUT1",
	            "Speaker", "LOUT2",
	            "Speaker", "ROUT2",
	            "Headphone", "Headphone Power",
	            "Headphone", "Headphone Power",
	            "Speaker", "Speaker Power",
	            "Speaker", "Speaker Power",
	            "LINPUT1", "Main Mic",
	            "LINPUT2", "Main Mic",
	            "RINPUT1", "Headset Mic",
	            "RINPUT2", "Headset Mic";
	        pinctrl-names = "default";
	        pinctrl-0 = <&hp_det>;
	        play-pause-key {
	            label = "playpause";
	            linux,code = <KEY_PLAYPAUSE>;
	            press-threshold-microvolt = <2000>;
	        };
	    };
	```
- **ES8388设备驱动配置**
	```bash
	&i2c7 {
	    status = "okay";
	    es8388: es8388@11 {
	        status = "okay";
	        #sound-dai-cells = <0>;
	        compatible = "everest,es8388", "everest,es8323";
	        reg = <0x11>;
	        clocks = <&cru I2S0_8CH_MCLKOUT>;
	        clock-names = "mclk";
	        assigned-clocks = <&cru I2S0_8CH_MCLKOUT>;
	        assigned-clock-rates = <12288000>;
	        pinctrl-names = "default";
	        pinctrl-0 = <&i2s0_mclk>;
	    };
	};
	```


## 2.3  编译烧写，调试

- **查看声卡命令：**`cat /proc/asound/cards`
	
	
- **将wav文件拷贝到板子上：**
	
	```bash
	adb root
	adb remount
	adb push C:\adb\test.wav mnt
	```
	
- **RK Android 播放音乐 ( RK Android SDK 标配 tiny-alsa 工具 ):**
	
	```bash
	adb shell
	cd /mnt
	tinyplay ./test.wav -D 0 -d 0
	```
	
-  **RK Android 录音：**
	
	```bash
	tinycap /sdcard/test.wav 
	
	播放录音
	cd /sdcard
	tinyplay ./test.wav -D 0 -d 0
	```
	
- **RK Linux 播放音乐 （ RK Linux SDK 标配 alsa-utils 工具 ）**
	
	```c
	 aplay  test.wav
	```
	**或者**
	```c
	aplay -Dplughw:0,0 test.wav
	
	aplay -Dplughw:1,0 test.wav
	
	aplay -Dplughw:2,0 test.wav
	
	-Dplughw:x  表示指定第几个声卡
	```
	**或者**
	```c
	aplay -D plughw:CARD=rockchipes8388 test.wav
	```
	
-  **RK Linux  录音**

	```c
	arecord -D hw:1,0 -d 10 -f cd -r 44100 -c 2 -t wav test.wav
	
	-d 10表示录制10秒声音，test.wav是保存的文件名称
	
	-D hw:x  表示指定第几个声卡
	
	-r 指定采样率，-f 指定每个采样点的位数--样本大小

	```

 ## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)