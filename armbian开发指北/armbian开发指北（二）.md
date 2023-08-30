# armbian开发指北（二）
前言：上篇给大家介绍armbian，已经初级入门用法，接下来会详细拆解开发步骤，本文讲带大家熟悉如何编译出自己想要的板级固件，armbian开发指北（三）会更加详细的带大家深入代码。
## 1. 编译
进入项目目录执行：
```
$ ./compile.sh
```
执行脚本后会提示

    [🚸] Docker is installed, but not usable [ can't use Docker; check your Docker config / groups / etc ]
    [💥] Problem detected [ Docker installed but not usable ]
    [💥] Exiting in 10 seconds [ Press <Ctrl-C> to abort, <Enter> to ignore and continue ]
    Counting down: 9... 8... 
armbian支持 docker环境编译，执行脚本后提示我们目前是docker环境是没有搭建好的，我们暂时不使用docker，按enter键即可

后提示输入权限密码 [sudo] password for jackson:

## 2. 选择是否更改kernel config

图片：armbian-choose-an-option.png

此时可看到两个选择

* Do not change the kernel configuration
* Show a kernel configuration menu before comilation

1. 选项为不更改kernel配置
2. 选项为在编译前可自行更改kernel配置

我们暂且先选择 Do not change the kernel configuration 不去更改内核配置，按enter键。

## 3. 选择目标

图片：armbian-select-target-board.png

大家下载下来的 armbian/build 应该是没有armsom-w3 和 armsom-p2pro, 是我们这边开发使用暂时没有合并提交到armbian官方仓库,后续会提交。

除此以上承列的板子都是目前官方支持的，我们可以选择自己需要编译的板子固件
,细心的朋友可能会注意到底部除了 ok，Cancel，还有一个 show CSC/WIP/EOS/TVB 选项

    CSC/WIP/EOS/TVB 解释：
    CSC: 社区支持的配置，社区贡献的支持。Armbian 开发团队没有官方支持
    WIP：正在进行中，基本功能可以测试，但尚未准备好投入生产。
    EOS：生命周期结束，支持结束。
    TVB：TV 电视盒子版本

我这边选择 armsom-w3 ：Rockchip RK3588 SoC octa core 8-32GB SoC 2.5GBe PoE eMMC USB3 NvME

## 4. 选择要使用什么kernel版本

图片：armbian-exact-kernel.png

如上我们可以看到Legacy版本，edge版本，midstream版本，collabora版本

    Legacy版本：旧的kernel版本，大概率是各个芯片厂家官方支持的kernel
    edge版本：最新主线的kernel
    midstream版本：介于主线mainline 和 rk官网legacy版本之间。
    collabora版本：Collabora's rk3588, where the action is these days

对于后面两个版本是开发者自己客制化的版本，可以自定义任何版本，armbian在这方面做的非常弹性。

选择自己想要编译的kernel版本，这边选legacy

## 5. 选择目标操作系统

图片：armbian-select-os.png

如上我们可以看到

* [bookworm Debian12 Bookworm](https://www.debian.org/releases/bookworm/)
* [bullseys Debian11 Bullseye](https://www.debian.org/releases/bullseye/) 
* [jammy Ubuntu jammy 22.04 LTS](https://releases.ubuntu.com/jammy/)

其实就是各种os版本，系统之间的差别可自行上网了解，选择你喜欢的，这边选择bullseye

## 6. 选择目标系统类型（是否带桌面系统）

图片：armbian-select-os-type.png

Image with console interface (server版本)
Image with desktop environment 桌面版本

开发验证阶段可用server版本编译的更快一些，需要验证显示接口建议用desktop版本

### 6.1 选择目标系统带桌面版本
假如选择desktop版本将会进入

图片: armbian-select-desktop-environment.png

cinnamon [Cinnamon desktopp environment](https://github.com/linuxmint/Cinnamon):

    Cinnamon 是一款 Linux 桌面，提供先进的创新功能和传统的用户体验。
    桌面布局类似于 Gnome 2，其底层技术源自 Gnome Shell。Cinnamon 提供易于使用且舒适的桌面体验，让用户有宾至如归的感觉。

gnome [Gnome desktop enviroment](https://www.gnome.org/):

    GNOME是一套纯粹自由的计算机软件，运行在操作系统上，提供图形桌面环境。
    桌面环境具有简洁、自定义性高、多任务支持、丰富的应用程序和开放性等特点，适合那些注重用户体验、个性化和开放性的用户。


i3-wm [I3-wm desktop enviroment](https://i3wm.org/)

    平铺式桌面实在是太棒了！也许你习惯了KDE、Gnome、Xfce、Cinammon这些主流桌面后可以尝试一下


xfce [Xfce desktop enviroment](https://www.xfce.org/)

    Xfce 是一个以速度、性能和资源效率为重点的轻量级桌面环境。它在不牺牲功能的情况下，提供了一个干净直观的用户界面。它采用了经过时间验证的、传统的图标和菜单驱动的用户界面，对提高生产力非常有效。此外，Xfce 还允许用户根据自己的偏好进行个性化设置。


我们偏好采用 gnome desktop，纯属个人喜好。

图片: armbian-select-desktop-software.png

之后会要求选择我们需要安装的软件，根据自己喜好选择，可多选。

### 6.2 选择目标系统server版本

图片: armbian-select-server-environment.png

Standard image with console interface：标准server版本

Minimal image with console interface：最小server版本

两者具体的差别，暂未深入

## 7. 总结

至此已经选择完所有配置开始编译，编译过程中大家可注意到

```
Repeat Build Options (early) [./compile.sh build BOARD=armsom-w3 BRANCH=legacy BUILD_DESKTOP=yes BUILD_MINIMAL=no DESKTOP_APPGROUPS_SELECTED='3dsupport browsers chat desktop_tools editors internet multimedia office programming remote_desktop' DESKTOP_ENVIRONMENT=gnome DESKTOP_ENVIRONMENT_CONFIG_NAME=config_base KERNEL_CONFIGURE=no RELEASE=bullseye ]
```

其中./compile.sh 后面的参数便是我们所有的选择，后续开发如果不想用图形页面选择，可直接输入以上命令
