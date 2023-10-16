# 1. 简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)
- 优秀的产品都要进行严苛的产品测试才能够经得起市场的检验
- 由ArmSoM团队研发的产测软件用于在量产的过程中快速地甄别产品功能和器件的好坏，即重点 FCT（Functional Test）测试，进而提高生产效率和检测的准确性。
- ArmSoM团队的专业产测软件用来保证量产的每一部产品的质量以及稳定性
- ArmSoM产测软件预览：
![在这里插入图片描述](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/armsom-test-image1.png)
# 2. 环境介绍


- 硬件环境：
ArmSoM-W3 RK3588开发板

- 软件版本：
OS：ArmSoM-W3 Debian11

# 3. ArmSom产测软件介绍
- QT开发的ARM平台产测图形化软件，一键开启傻瓜式测试

- ArmSoM产测软件是基于Linux和Android平台：Ubantu和Debian，Android系统都适用

- ArmSoM产测软件是直接安装在RK3588开发板上，接上屏幕即可打开产测软件进行测试。

- ArmSoM产测软件是由本公司开发，现已应用于商业量产产测。
# 4. 技术要点：
- 线程池实现的多线程技术，全部接口功能并行同时测试，极大的提高了产测效率

- qt + opencv实现的Camera，Hdmiin实时显示视频画面。

- 接口功能测试代码编写，准确率高，精准定位接口功能的好坏

- 可支持多款开发板进行产测。用户可手动选择需要测试的开发板

- 支持多平台，支持各种架构的开发板测试，不局限于ARM开发板

- 可扩展性高，量身定制需要测试的功能接口，支持扩展全功能接口测试

# 5. 目前支持的测试项接口
- 测试项目包括自动测试项和手动测试项

- 自动测试项目无需人工干预测试结束后会直接上报测试结果并显示通过与否
- 人工测试项目需要人为判断测试项是否正确完成，并给出判断（通过或不通过）。

- 目前支持：WIFI 测试、蓝牙测试、USB 测试、LED灯测试、放音测试、录音测试、Camera 测试,Hdmi-in测试，40PIN测试，网口测试，M2接口测试，RTC测试等等

- 可扩展性高，量身定制需要测试的功能接口，支持扩展全功能接口测试。

# 6. 产测视频：



[video(video-RCHLuVin-1685618386502)(type-csdn)(url-https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/armsom-test-video.mp4)(title-产测视频.)]

## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)