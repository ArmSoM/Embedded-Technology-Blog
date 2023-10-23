# 1. 简介
- [专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- ArmSoM团队在产品量产之前都会对产品做几次专业化的功能测试以及性能压力测试，以此来保证产品的质量以及稳定性

- 优秀的产品都要进行多次全方位的功能测试以及性能压力测试才能够经得起市场的检验

# 2. ArmSoM-W3软硬件重启测试方案
- 软件方式重启系统3000次测试
- 硬件电源拔插重启3000次测试
# 3. 软件重启3000次测试
- 测试原理：对目标板进行3000次软件方式重启系统测试，看开发板运行情况，是否能扛起3000次的连续重启。
- 测试时间：2023年5月7日 9:55 -- 5月8日 13:50
- 测试工具：RK3588 - ArmSoM-W3开发板，电源，屏幕，HDMI线，鼠标
- 测试步骤：
	
	
	> 1. 准备5块ArmSoM-W3开发板，全部烧写ArmSoM-W3-Box版本的固件。
	> 2. 开发板均接入串口，保存开关机时串口打印的log。查看开发板实际开关机多少次。查看开发板重启期间是否异常。
	> 	3. 使用专业化测试软件reboot test应用程序  设置重启次数：3000次。然后点击start开始测试。

- 测试结果：进行了3000次软件方式重启，5块开发板均运行正常。

![reboot-testing-software](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/reboot-testing-software.jpeg#pic_left=600x)

# 4. 硬件重启3000次测试
- 测试原理：对目标板进行3000次电源拔插测试，看开发板运行情况，是否能扛起3000次的连续硬件重启。
- 测试时间：2023年5月8日 17:52 -- 5月10日9:02

- 测试工具：串口，电脑，RK3588 - ArmSoM-W3开发板，两个定时器（设定时间自动断电源，开电源）

- 测试软件:   MobaXterm软件记录测试时段的log打印，并将log数据保存好。

- 测试步骤：

	> 1. 准备5块ArmSoM-W3开发板，接好定时器电源，串口连接电脑。
	> 2. 打开MobaXterm软件记录测试时段的log打印
	> 3. 设定开发板的启动时间25秒和关机时间10秒。
	> 4. 根据log打印查看RK3588 - ArmSoM-W3开发板实际开关机多少次，查看开发板拔插电源期间是否异常。

- 测试结果：进行了3000次硬件断电源方式重启，开关机次数相差一两次在误差范围内，硬件重启测试未发现异常。

![reboot-testing-hardware](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/reboot-testing-hardware.jpeg#pic_left=600x)