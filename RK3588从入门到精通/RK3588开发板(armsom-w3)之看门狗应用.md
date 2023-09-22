本篇将对看门狗使用进行学习

<img src="C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588开发板(armsom-w3)之看门狗应用\33333333.jpg" style="zoom:33%;" />



# 前言

​		看门狗（watchdog）实际是一个定时器，当一个硬件系统开启了watchdog功能，那么运行在这个硬件系统之上的软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如此反复下去，以此来确定系统和软件正常运行。



# 一、什么是看门狗

​		watchdog，中文名称叫做'看门狗"，全称watchdog timer，简称WDT;它属于一种定时器，然而它与我们平常所接触的定时器在作用上又有所不同。
看门狗，一般有一个输入和一个输出。在设备正常工作的时候，每隔一段时间输出一个信号到一个寄存器，如果在超过规定的时间没有输出信号，看门狗会把寄存器数据清零。如果规定时间内没有喂狗，看门狗超时，说明系统或应用陷入循环、卡死，此时看门狗会发出复位信号让主控复位，让设备进入正常工作状态。ArmSom-W3开发板上有 1 个内部看门狗 ，之后章节主要介绍该看门狗的使用。



# 二、WDT 驱动

1、watchdog 的驱动文件为 `kernel-5.10/drivers/watchdog/dw_wdt.c`

2、ArmSom-W3的 watchdog 的 DTS 节点在 `kernel/arch/arm64/boot/dts/rockchip/rk3588s.dtsi` 文件中定义，如下所示：

```
wdt: watchdog@feaf0000 {
    compatible = "snps,dw-wdt";
    reg = <0x0 0xfeaf0000 0x0 0x100>;
    clocks = <&cru TCLK_WDT0>, <&cru PCLK_WDT0>;
    clock-names = "tclk", "pclk";
    interrupts = <GIC_SPI 315 IRQ_TYPE_LEVEL_HIGH>;
    status = "disabled";
};
```

用户首先需在 DTS 文件中打开 wdt 节点：

```
&wdt{
    status = "okay";
};
```

DTS 配置参考文档 为 Documentation/devicetree/bindings/watchdog/dw_wdt.txt，主要说明如下参数:

- interrupts = <GIC_SPI 120 IRQ_TYPE_LEVEL_HIGH 0>;

  中断模式时候用于首先触发中断，再经过一个超时周期才产生复位信号。

- clocks = <&cru PCLK_WDT>;

  驱动WDT工作，并且用于计算每个计数周期。

3、内核配置文件是在\kernel\arch\arm64\configs\armsom_w3_defconfig

```
CONFIG_WATCHDOG=y
CONFIG_DW_WATCHDOG=y
```

4、驱动默认的超时时间在dw_wdt.c里面有描述

```
/* There are sixteen TOPs (timeout periods) that can be set in the watchdog. */
#define DW_WDT_NUM_TOPS		16
#define DW_WDT_DEFAULT_SECONDS	30
```

​		驱动中看门狗默认设置的时间是DW_WDT_DEFAULT_SECONDS=30秒，但允许再超时16秒执行DW_WDT_MAX_TOPS=16秒,
所以总的超时时间为46秒，46秒后还没进行喂狗，设备将重启;

5、内核中watchdog默认是不开启的，需要开启的话，需APP应用或者开启一个服务去喂狗即可开启watchdog功能watchdog设备节点为:/dev/watchdog 或者/dev/watchdog0



# 三、Watchdog 使用

下面介绍两种方法来使用 watchdog，一是通过程序来控制看门狗，二是通过 `echo` 命令来控制该设备。

## 3.1 内核接口

相关函数定义在`kernel/include/linux/watchdog.h`和`/kernel/include/uapi/linux/watchdog.h`
![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588开发板(armsom-w3)之看门狗应用\1.png)

- options 字段记录了设备支持哪些功能或选项
- firmware_version 字段记录了设备的固件版本号
- identity 字段则是一个描述性的字符串

重点关注的是 options 字段，该字段描述了设备支持哪些功能、选项。

​		 应用层控制看门狗其实非常简单，通过 ioctl()函数即可做到，上述头文件中定义了一些 ioctl 指令宏，每一个不同的指令宏表示向设备请求不同的操作。
比 较 常 用 指 令如下：

1. WDIOC_GETSUPPORT	获取看门狗支持哪些功能
2. WDIOC_SETOPTIONS	用于开启或关闭看门狗
3. WDIOC_KEEPALIVE	喂狗操作
4. WDIOC_SETTIMEOUT	设置看门狗超时时间
5. WDIOC_GETTIMEOUT	获取看门狗超时时间


对于文件操作，在`kernel/drivers/watchdog/watchdog_dev.c`中有定义

```
static const struct file_operations watchdog_fops = {
	.owner		= THIS_MODULE,
	.write		= watchdog_write,
	.unlocked_ioctl	= watchdog_ioctl,
	.compat_ioctl	= compat_ptr_ioctl,
	.open		= watchdog_open,
	.release	= watchdog_release,
};
```

其中需要关注的接口是watchdog_ioctl

```
static long watchdog_ioctl(struct file *file, unsigned int cmd,
							unsigned long arg)
{
	struct watchdog_core_data *wd_data = file->private_data;
	void __user *argp = (void __user *)arg;
	struct watchdog_device *wdd;
	int __user *p = argp;
	unsigned int val;
	int err;

	mutex_lock(&wd_data->lock);

	wdd = wd_data->wdd;
	if (!wdd) {
		err = -ENODEV;
		goto out_ioctl;
	}

	err = watchdog_ioctl_op(wdd, cmd, arg);
	if (err != -ENOIOCTLCMD)
		goto out_ioctl;

	switch (cmd) {
	case WDIOC_GETSUPPORT:
		err = copy_to_user(argp, wdd->info,
			sizeof(struct watchdog_info)) ? -EFAULT : 0;
		break;
	case WDIOC_GETSTATUS:
		val = watchdog_get_status(wdd);
		err = put_user(val, p);
		break;
	case WDIOC_GETBOOTSTATUS:
		err = put_user(wdd->bootstatus, p);
		break;
	case WDIOC_SETOPTIONS:
		if (get_user(val, p)) {
			err = -EFAULT;
			break;
		}
		if (val & WDIOS_DISABLECARD) {
			err = watchdog_stop(wdd);
			if (err < 0)
				break;
		}
		if (val & WDIOS_ENABLECARD)
			err = watchdog_start(wdd);
		break;
	case WDIOC_KEEPALIVE:
		if (!(wdd->info->options & WDIOF_KEEPALIVEPING)) {
			err = -EOPNOTSUPP;
			break;
		}
		err = watchdog_ping(wdd);
		break;
	case WDIOC_SETTIMEOUT:
		if (get_user(val, p)) {
			err = -EFAULT;
			break;
		}
		err = watchdog_set_timeout(wdd, val);
		if (err < 0)
			break;
		/* If the watchdog is active then we send a keepalive ping
		 * to make sure that the watchdog keep's running (and if
		 * possible that it takes the new timeout) */
		err = watchdog_ping(wdd);
		if (err < 0)
			break;
		fallthrough;
	case WDIOC_GETTIMEOUT:
		/* timeout == 0 means that we don't know the timeout */
		if (wdd->timeout == 0) {
			err = -EOPNOTSUPP;
			break;
		}
		err = put_user(wdd->timeout, p);
		break;
	case WDIOC_GETTIMELEFT:
		err = watchdog_get_timeleft(wdd, &val);
		if (err < 0)
			break;
		err = put_user(val, p);
		break;
	case WDIOC_SETPRETIMEOUT:
		if (get_user(val, p)) {
			err = -EFAULT;
			break;
		}
		err = watchdog_set_pretimeout(wdd, val);
		break;
	case WDIOC_GETPRETIMEOUT:
		err = put_user(wdd->pretimeout, p);
		break;
	default:
		err = -ENOTTY;
		break;
	}

out_ioctl:
	mutex_unlock(&wd_data->lock);
	return err;
}
```

## 3.2 操控看门狗

**打开看门狗设备**：

```
open("/dev/watchdog", "O_RDWR")
```

需要注意的是，当调用 open()打开看门狗设备的时候， 即使程序中没有开启看门狗计时器，所以，当打开设备之后， 需要使用 指令停止看门狗计时，等所有设置完成之后再开启看门狗计时器

**开启/关闭看门狗**：
使用 WDIOC_SETOPTIONS 指令可以开启看门狗计时或停止看门狗计时，使用方式如下：

```
ioctl(int fd, WDIOC_SETOPTIONS, int *option);
```

 option 指针指向一个 int 类型变量，该变量可取值如下：

```
#define WDIOS_DISABLECARD 0x0001 /*Turn off the watchdog timer停止看门狗计时*/
#define WDIOS_ENABLECARD 0x0002 /* Turn on the watchdog timer开启看门狗计时*/
```

 **获取/设置超时时间**:
使用 WDIOC_GETTIMEOUT 指令可获取设备当前设置的超时时间，使用方式如下：

```
ioctl(int fd, WDIOC_GETTIMEOUT, int *timeout);
```

 使用 WDIOC_SETTIMEOUT 指令可设置看门狗的超时时间，使用方式如下：

```
ioctl(int fd, WDIOC_SETTIMEOUT, int *timeout);
```

 超时时间是以秒为单位， 设置超时时间时，不可超过其最大值、否则 ioctl()调用将会失败

**喂狗**： 

看门狗计时器启动之后，需要在超时之前，去“喂狗”，否则计时器溢出超时将会导致系统复位或产生一个中断信号，通过 WDIOC_KEEPALIVE 指令喂狗，使用方式如下：

```
ioctl(int fd, WDIOC_KEEPALIVE, NULL);
```

下面是一个例程可供参考：

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/watchdog.h>

// 定义看门狗设备信息结构体
struct watchdog_info wd_info;
// 超时时间（秒）
int timeout = 5;

int main(int argc, char *argv[]) {
    // 检查命令行参数
    if (argc != 3 || strcmp(argv[1], "runapp") != 0) {
        printf("Usage: %s runapp <timeout>\n", argv[0]);
        exit(1);
    }

    // 获取超时时间
    timeout = atoi(argv[2]);
    if (timeout < 5) {
        printf("Timeout value too short. Using default of 5 seconds.\n");
        timeout = 5;
    }

    // 打开看门狗设备
    int fd = open("/dev/watchdog", O_RDWR);
    if (fd == -1) {
        perror("Failed to open watchdog device");
        exit(1);
    }

    // 关闭看门狗计时，防止立即重启系统
    ioctl(fd, WDIOC_KEEPALIVE, 0);

    // 设置超时时间
    if (ioctl(fd, WDIOC_SETTIMEOUT, &timeout) != 0) {
        perror("Failed to set watchdog timeout");
        close(fd);
        exit(1);
    }

    printf("Watchdog started with a timeout of %d seconds.\n", timeout);

    // 设置喂狗时间为超时前1秒
    int feed_time = timeout - 1;

    // 开始看门狗计时并喂狗
    while (1) {
        sleep(feed_time);
        printf("Feeding the watchdog...\n");
        ioctl(fd, WDIOC_KEEPALIVE, 0);
    }

    // 关闭看门狗设备
    close(fd);
    return 0;
}
```

## 3.3 命令操作watchdog

开启watchdog功能，只需要往/dev/watchdong写入除了大写“V”字符外，其他任意字符都可以，比如：

```
root@linaro-alip:/# echo x > /dev/watchdog
[34818.074700] watchdog: watchdog0: watchdog did not stop!
```

这是测试命令，所以在46秒后系统会自动重启，但是在这期间喂狗就不会自动重启了，例如写入大写字符”V“

```
echo V > /dev/watchdog
```

写入大写字符“V”，内核会自动喂狗，但是系统不会自动重启。

**测试阶段可以让系统模拟死机**
在不断电源的情况下，让系统处于永久死机的状态，可以让内核发生panic，将`/proc/sys/kernel/panic`设置为0，表示在内核发生严重错误时将重新引导。Oops由于内核引用了无效指针;发生于用户空间程序通常产生一个段错误segfault，而用户态程序自身无法恢复;发生于内核空间时则称作oops。
往oops节点写入c字符即可让系统死机。
这里可以手动设置：

```
echo 0 > /proc/sys/kernel/panic
echo c > /proc/sysrq-trigger
```

执行上面两条命令之后，kernel就直接崩溃了，卡在这个界面：

![](C:\Users\Administrator\Desktop\撰写文档\csdn\md文档\RK3588从入门到精通\RK3588开发板(armsom-w3)之看门狗应用\222222.png)

如果死机之前没有开启watchdog，在不断开电源的情况下，机器永久死机。但是在死机前有开启watchdog功能，等喂狗时间超时，系统会自动重启。