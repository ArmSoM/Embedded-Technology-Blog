# 1. 简介
- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)

- ArmSoM团队在产品量产之前都会对产品做几次专业化的功能测试以及性能压力测试，以此来保证产品的质量以及稳定性

- 优秀的产品都要进行多次全方位的功能测试以及性能压力测试才能够经得起市场的检验
# 2. 环境介绍

- 硬件环境：
ArmSoM-W3 RK3588开发板

- 软件版本：
OS：ArmSoM-W3 Debian11
![在这里插入图片描述](https://img-blog.csdnimg.cn/69e7c5eda91649d7b8a4d17e6a564c4d.jpeg#pic_left =600x)
# 3. ArmSoM-W3 DDR压力测试方案
**测试方案：同时对DDR进行三项压力测试：**
- 使用memtester工具对DDR进行压力测试
- 使用stressapptest工具对DDR进行压力测试
- 使用RK官方测试脚本进行DDR变频测试

# 4.DDR压力测试
- 测试原理：运行RK官方的DDR压力测试脚本，同时对DDR进行三项压力测试，看开发板运行情况，是否能扛起24小时的连续DDR压力测试。
- 测试时间：2023年8月31日 9:55 -- 9月1日 10:02
- 测试工具：RK3588 - ArmSoM-W3开发板，电源，屏幕，HDMI线，鼠标，串口
- 测试步骤：
	
	
	> 1. 准备5块ArmSoM-W3开发板，全部烧写ArmSoM-W3-Debian版本的固件。
	> 2. 开发板均接入串口，保存压力测试时串口打印的log。查看开发板在DDR压力测试期间是否异常。
	>   3. 运行RK官方的DDR压力测试脚本，同时对DDR进行三项压力测试

- 测试结果：进行了24小时的DDR压力测试，5块开发板均运行正常。


# 5. DDR压力测试流程
- 串口输入命令进入rockchip-test测试目录：

	```c
	root@linaro-alip:/# cd /rockchip-test
	```
- 运行rockchip-test测试脚本：选择 1：ddr stress test    继续选择5：stressapptest + memtester + ddr auto scaling

	```c
	root@linaro-alip:/rockchip-test# ./rockchip_test.sh
	```
	![在这里插入图片描述](https://img-blog.csdnimg.cn/820846db63904bc48aea608485872a70.png)

- DDR压力测试开始：![在这里插入图片描述](https://img-blog.csdnimg.cn/a0c4025e5de843ce8506b08537c7bdc1.png)

	![在这里插入图片描述](https://img-blog.csdnimg.cn/f8e83fa157ef4480966cf46c53df57c7.jpeg#pic_left =600x)
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc53a2ab3b2b480c9b3b873608e0fb52.jpeg#pic_left =600x)
## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)