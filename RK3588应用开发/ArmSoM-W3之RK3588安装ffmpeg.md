# 1. 简介
- FFmpeg 是一个完整的、跨平台的音频和视频录制、转换和流媒体解决方案。既是一款音视频编解码工具，同时也是一组音视频编解码开发套件，作为编解码开发套件，它为开发者提供了丰富的音视频处理的调用接口。


- FFmpeg 提供了多种媒体格式的封装和解封装，包括多种音视频编码，多种协议的流媒体，多种色彩格式转换，多种采样率转换，多种码率转换等。ffmpeg发展至今，已经被许多开源项目使用。
- FFmpeg 官网：[http://ffmpeg.org/](http://ffmpeg.org/)
- 本文介绍RK3588平台安装ffmpeg

# 2. 环境介绍

- 硬件环境：
ArmSoM-W3 RK3588开发板

- 软件版本：
OS：ArmSoM-W3 Debian11
# 3. ffmpeg 4.3.1 安装

## 3.1下载：

```bash
wget http://www.ffmpeg.org/releases/ffmpeg-4.3.1.tar.gz
```

```bash
tar -xvf ffmpeg-4.3.1.tar.gz

cd ffmpeg-4.3.1/

./configure --prefix=/usr/local/my/ffmpeg --enable-version3 --enable-rkmpp --enable-nonfree --enable-gpl --enable-shared

make -j8
sudo make install
```
## 3.2 然后更改配置文件/etc/ld.so.conf

```bash
sudo vim /etc/ld.so.conf

include /etc/ld.so.conf.d/*.conf
#复制下面内容
/usr/local/lib #librockchip_mpp.so
```
然后执行sudo ldconfig命令生效
将ffmpeg路经添加到PATH

```bash
sudo vim .bashrc
#最后一行添加自己的ffmpeg路经
export PATH=$PATH:/usr/local/my/ffmpeg/bin
```
然后执行source .bashrc生效
查看一下系统PATH，可以看到已经将ffmpeg添加好了
```bash
echo $PATH
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8745b16ce3704eef9c212c60be9d17f3.png)
## 3.3 检查是否成功安装

```bash
ffmpeg -version
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/78e8446c321e4913b5f19c104a78fc86.png)
# 4.  卸载旧的ffmeg
想要重新安装的话，要先卸载ffmeg

```bash
sudo apt-get --purge remove ffmpeg
sudo apt-get --purge autoremove
```
如果你使用的是总网上下载安装包，然后编译安装的方法，则需要使用以下的方式卸载，此处以ffmpeg- 4.3.1为例：


```bash
cd ffmpeg-4.3.1
make uninstall  ##删除由make install命令安装的文件
make clean  ##只删除make时产生的临时文件
make distclean  ##同时删除configure和make产生的临时文件
```