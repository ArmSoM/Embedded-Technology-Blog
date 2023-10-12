# 1. 简介
- [专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- ArmSoM团队在产品量产之前都会对产品做几次专业化的功能测试以及性能压力测试，以此来保证产品的质量以及稳定性

- 优秀的产品都要进行多次全方位的功能测试以及性能压力测试才能够经得起市场的检验
# 2. 环境介绍

- 硬件环境：
ArmSoM-W3 RK3588开发板

- 软件版本：
OS：ArmSoM-W3 Debian11
# 3. ArmSoM-W3 DDR带宽测试方案
- rk-msch-probe-for-user是官方提供的用于统计和监控系统DDR的负载和带宽使用情况的工具，可以实时显示当前DDR的负载和带宽信息。
- 使用rk-msch-probe-for-use工具统计和监控系统DDR的负载和带宽使用情况
# 4. DDR带宽测试
- 测试原理：运行RK官方的DDR带宽测试工具，统计和监控系统DDR的负载和带宽使用情况
- 测试时间：2023年10月11日
- 测试工具：RK3588 - ArmSoM-W3开发板，电源，屏幕，HDMI线，鼠标，串口
## 4.1 测试步骤：


1. rk-msch-probe-for-user工具需要在定频的模式下才能使用
	设置DDR定频在最高频率2112MHz

	```bash
	//切换到用户空间
	root@linaro-alip:/# echo userspace > sys/class/devfreq/dmc/governor
	
	//获取系统支持的频点信息
	root@linaro-alip:/# cat sys/class/devfreq/dmc/available_frequencies
	528000000 1068000000 1560000000 2112000000
	
	//设置DDR定频在最高频率2112MHz
	root@linaro-alip:/# echo 2112000000 > sys/class/devfreq/dmc/userspace/set_freq
	```
2. 修改rk-msch-probe-for-use工具权限为777

	```bash
	chmod 777 ./data/rk-msch-probe-for-user-64bit
	```

3. 开始运行

	```bash
	./data/rk-msch-probe-for-user-64bit -c rk3588
	```

	
	```bash
	root@linaro-alip:/# ./data/rk-msch-probe-for-user-64bit -c rk3588
	V1.44_20230928
	
	2kijec4hi======================================================================================================
	ddr freq: 2112Mhz          cpu      vicap        gpu        vop        isp     others      total
	master bw(MB/s)           0.64       0.00       0.00    1019.79       0.00      24.79    1045.22
	bw prorated(%)            0.06       0.00       0.00      97.57       0.00       2.37     100.00
	utilization(%)            0.00       0.00       0.00       3.02       0.00       0.07       3.09
	----------------------------------------------ALL-------------------------CH0-------------------------CH1-------------------------CH2-------------------------CH3--------
	               recorded LOAD: max 1045.22MB/s(3.09%), min 1045.22MB/s(3.09%), avg 1045.22MB/s(3.09%)
	                        LOAD:         1045.22MB/s(3.09%),          261.50MB/s(3.10%),          261.24MB/s(3.09%),          261.18MB/s(3.09%),          261.31MB/s(3.09%)
	                          RD:         1045.16MB/s(3.09%),          261.46MB/s(3.09%),          261.23MB/s(3.09%),          261.17MB/s(3.09%),          261.30MB/s(3.09%)
	                          WR:            0.07MB/s(0.00%),            0.04MB/s(0.00%),            0.01MB/s(0.00%),            0.01MB/s(0.00%),            0.01MB/s(0.00%)
	-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
	=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
	```
4. 设备上运行需要监控ddr信息的应用，实时监控ddr的带宽使用情况。

## 4.2 测试统计的结果说明
 由上图的测试结果得出： 	在监控时间的1000ms中：所有channel的平均带宽为1045.22MB/s，负载为3.09%。




	>   ALL：	所有channel总的带宽统计信息
	>   CHx：    DDR channel x的带宽统计信息
	>   LOAD：   对于所有DDR bank，此channel的带宽及负载
	>   RD：     对于所有DDR bank，DDR read 数据的带宽及占比 
	>   WR：	    对于所有DDR bank，DDR write 数据的带宽及占比


## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)