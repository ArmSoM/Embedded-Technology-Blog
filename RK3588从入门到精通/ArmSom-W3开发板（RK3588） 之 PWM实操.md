# PWM介绍

- [专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- PWM是脉宽调制（Pulse Width Modulation）的缩写。它是一种用于控制电子设备的技术，通过改变电信号的脉冲宽度来实现对设备的控制。

## PWM基本概念


PWM信号由一个固定频率的周期性脉冲序列组成，每个脉冲的宽度（持续时间）可以根据需要进行调节。调节脉冲宽度的比例可以改变平均电压或电流的大小，从而实现对设备的控制。

当谈论PWM时，以下三个关键术语经常被提及：
频率（Frequency）：PWM信号的频率是指每秒钟内脉冲的数量。
周期（Period）：PWM信号的周期是指一个完整脉冲序列所花费的时间。它是频率的倒数，以秒为单位表示。周期可以通过将频率的倒数计算得到，例如，一个10kHz的PWM信号的周期为0.1毫秒（100微秒）。
占空比（Duty Cycle）：占空比是指PWM信号中脉冲宽度与周期之间的比例关系。它表示了脉冲在一个周期中所占据的时间比例，通常以百分比表示。占空比为0%意味着脉冲不存在（完全低电平），而占空比为100%表示脉冲持续时间占据了整个周期（完全高电平）。在实际应用中，占空比可以在0%到100%之间任意调整，以实现所需的控制效果。

# PWM驱动
pwm驱动是一个通用的驱动，SOC厂家都会在SDK里面默认打开

## 驱动文件

驱动文件所在位置：

> drivers/pwm/pwm-rockchip.c

默认SDK已经加载好了PWM的驱动，下文我们主要注意PWM怎么使用

## DTS 节点配置

DTS 配置参考文档 

> Documentation/devicetree/bindings/pwm/pwm.txt

以下为一个例子的示例

```c
Node name { 
	compatible = "Driver matching character"; 
	pwms = <&pwmX 0 25000 0>; 
}; 
&pwmX { 
	status = "okay"; 
	pinctrl-names = "active"; 
	pinctrl-0 = <&pwmX_pin_pull_down>; 
};
```
pwms的几个参数说明如下：
参数 1，表示 index (per-chip index of the PWM to request)，一般是 0，因为我们 Rockchip PWM 每个chip 只有一个。
参数 2，表示 PWM 输出波形的时间周期，单位是 ns；例如下面配置的 25000 就是表示想要得到的
PWM 输出周期是 40K 赫兹。
参数 3，表示极性，为可选参数；下面例子中的配置为负极性。


# PWM使用

PWM 提供了用户层的接口，在 /sys/class/pwm/ 节点下面，PWM 驱动加载成功后，会在/sys/class/pwm/ 目录下产生 pwmchip0 目录；向 export 文件写入 0，就是打开 pwm 定时器 0，会产生一个 pwm0 目录，相反的往 unexport 写入 0 就会关闭 pwm 定时器了，同时 pwm0 目录会
被删除,该目录下有以下几个文件：

> enable：写入 1 使能 pwm，写入 0 关闭 pwm；
>  polarity：有 normal 或 inversed两个参数选择，表示输出引脚电平翻转； 
>  duty_cycle：在 normal 模式下，表示一个周期内高电平持续的时间（单位：纳秒），在
> reversed 模式下，表示一个周期中低电平持续的时间（单位：纳秒)； 
> period：表示 pwm 波的周期(单位：纳秒)；

以下是 pwmchip0 的例子，设置 pwm0 输出频率 100K，占空比 50%, 极性为正极性：

```c
cd /sys/class/pwm/pwmchip0/
echo 0 > export
cd pwm0
echo 10000 > period
echo 5000 > duty_cycle
echo normal > polarity
echo 1 > enable
```

## PWM应用实例
通常电子设备中应用pwm是比较常见的，比如风扇电机控制，电视背光控制， LED 照明调光、电动工具马达控制、汽车加热器等领域。

这里简单介绍一下pwm控制LED灯实现呼吸灯效果。
呼吸灯需要灯的驱动与PWM的驱动结合，两个驱动之间传递数据，我们可以在驱动中调用其他的驱动。
led是我需要的设备，这个设备用到了pwm，而pwm是用默认的驱动。
硬件上我们在开发板找到具有pwm功能的引脚
![ArmSoM-W3_40pin](https://img-blog.csdnimg.cn/23eb9ffe364348a4aa3dc9b86d0770ea.png)
![pwm8](https://img-blog.csdnimg.cn/38a3fef1095842d7a359f43682cf7d3e.png)


设备树的修改如下：

```c
/{
	breathing_light {
		compatible = "lhd,breathing_light_test";
		backlight {
			pwms = <&pwm8 0 25000 0>;
			pwm-names = "breathing_light"; 
		};
	};
};
&pwm8 {
	status = "okay";
};
```
写一个驱动。内部在使用PWM子系统。形成了包含驱动的驱动。
## 示例代码
驱动程序
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/types.h>
#include <linux/kernel.h>
#include <linux/delay.h>
#include <linux/ide.h>
#include <linux/errno.h>
#include <linux/gpio.h>
//#include <asm/mach/map.h>
#include <linux/of.h>
#include <linux/of_address.h>
#include <linux/of_gpio.h>
#include <asm/io.h>
#include <linux/device.h>
#include <linux/platform_device.h>
#include <linux/pwm.h>

#define RED_LED_DTS_COMPATIBLE       "lhd,breathing_light_test"       /* 设备树节点匹配属性 */

#define LED_PWM_CMD_SET_DUTY         0x01
#define LED_PWM_CMD_SET_PERIOD       0x02
#define LED_PWM_CMD_SET_BOTH         0x03
#define LED_PWM_CMD_ENABLE           0x04
#define LED_PWM_CMD_DISABLE          0x05

struct led_pwm_param {
    int duty_ns;
    int period_ns;
};

struct red_led_dev {
    dev_t dev_no;                    
    struct cdev chrdev;            
    struct class *led_class;
    struct device_node *dev_node;
    struct pwm_device *red_led_pwm;
};

static struct led_pwm_param led_pwm;
static struct red_led_dev led_dev;

static int red_led_drv_open (struct inode *node, struct file *file)
{
    int ret = 0;
    
	//pwm_set_periodnnn(led_dev.red_led_pwm, PWM_POLARITY_INVERSED);//设置PWM信号的极性
	pwm_enable(led_dev.red_led_pwm);//启用指定PWM设备，使其开始输出PWM信号。

    printk("red_led_pwm open\r\n");
    return ret;
}

static ssize_t red_led_drv_write (struct file *file, const char __user *buf, size_t size, loff_t *offset)
{
    int err;

    if (size != sizeof(led_pwm)) return -EINVAL;

	err = copy_from_user(&led_pwm, buf, size);
    if (err > 0) return -EFAULT;

	pwm_config(led_dev.red_led_pwm, led_pwm.duty_ns, led_pwm.period_ns);//配置PWM设备的基本参数，如频率、占空比等。
    printk("red_led_pwm write\r\n");
	return 1;
}

static long drv_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
    int ret = 0;
    void __user *my_user_space = (void __user *)arg;
    
    switch (cmd)
    {
        case LED_PWM_CMD_SET_DUTY:
            ret = copy_from_user(&led_pwm.duty_ns, my_user_space, sizeof(led_pwm.duty_ns));
            if (ret > 0) return -EFAULT;
            pwm_config(led_dev.red_led_pwm, led_pwm.duty_ns, led_pwm.period_ns);
            break;
        case LED_PWM_CMD_SET_PERIOD:
            ret = copy_from_user(&led_pwm.period_ns, my_user_space, sizeof(led_pwm.period_ns));
            if (ret > 0) return -EFAULT;
            pwm_config(led_dev.red_led_pwm, led_pwm.duty_ns, led_pwm.period_ns);
            break;
        case LED_PWM_CMD_SET_BOTH: 
            ret = copy_from_user(&led_pwm, my_user_space, sizeof(led_pwm));
            if (ret > 0) return -EFAULT;
            pwm_config(led_dev.red_led_pwm, led_pwm.duty_ns, led_pwm.period_ns);
            break;
        case LED_PWM_CMD_ENABLE:
            pwm_enable(led_dev.red_led_pwm);
            break;
        case LED_PWM_CMD_DISABLE:
            pwm_disable(led_dev.red_led_pwm);
            break;
    }
    return 0;
}

static int red_led_drv_release(struct inode *node, struct file *filp)
{
    int ret = 0;

    pwm_config(led_dev.red_led_pwm, 0, 5000);//配置PWM设备的基本参数，如频率、占空比等。
    printk("led pwm dev close\r\n");
//    pwm_disable(led_dev.red_led_pwm);
    return ret;
}

static struct file_operations red_led_drv = {
	.owner	 = THIS_MODULE,
	.open    = red_led_drv_open,
	.write   = red_led_drv_write,
    .unlocked_ioctl = drv_ioctl,
    .release  = red_led_drv_release,
};

/*设备树的匹配列表 */
static struct of_device_id dts_match_table[] = {
    {.compatible = RED_LED_DTS_COMPATIBLE, },  
    {},                  
};


static int led_red_driver_probe(struct platform_device *pdev)
{
    int err;
    int ret;
    struct device *tdev;
    struct device_node *child;

    tdev = &pdev->dev;
    child = of_get_next_child(tdev->of_node, NULL);      /* 获取设备树子节点 */
	if (!child) {
        return -EINVAL;
    }

    led_dev.red_led_pwm = devm_of_pwm_get(tdev, child, NULL);     /* 从子节点中获取PWM设备，设备树获取这个设备就可以了 */
    if (IS_ERR(led_dev.red_led_pwm)) {
        printk(KERN_ERR"can't get breathing_light!!\n");
        return -EFAULT;
    }

    ret = alloc_chrdev_region(&led_dev.dev_no, 0, 1, "breathing_light");//动态分配字符设备的主设备号
	if (ret < 0) {
		pr_err("Error: failed to register mbochs_dev, err: %d\n", ret);
		return ret;
	}

	cdev_init(&led_dev.chrdev, &red_led_drv);//初始化字符设备结构体cdev

	cdev_add(&led_dev.chrdev, led_dev.dev_no, 1);//将已经初始化的字符设备结构体cdev添加到系统中

    led_dev.led_class = class_create(THIS_MODULE, "breathing_light");//创建一个设备类（device class）并注册到内核中
	err = PTR_ERR(led_dev.led_class);
	if (IS_ERR(led_dev.led_class)) {
        goto failed1;
	}

    tdev = device_create(led_dev.led_class , NULL, led_dev.dev_no, NULL, "breathing_light"); //创建一个设备实例并注册到设备类中
    if (IS_ERR(tdev)) {
        ret = -EINVAL;
		goto failed2;
	}

   	printk(KERN_INFO"%s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);
    
    return 0;
failed2:
    device_destroy(led_dev.led_class, led_dev.dev_no);
    class_destroy(led_dev.led_class);
failed1:
    cdev_del(&led_dev.chrdev);
	unregister_chrdev_region(led_dev.dev_no, 1);
    return ret;
}

int led_red_driver_remove(struct platform_device *dev)
{
    // pwm_disable(led_dev.red_led_pwm);
    // pwm_free(led_dev.red_led_pwm);
    printk(KERN_INFO"driver remove %s %s line %d\n", __FILE__, __FUNCTION__, __LINE__);
    device_destroy(led_dev.led_class, led_dev.dev_no);
	class_destroy(led_dev.led_class);
	unregister_chrdev_region(led_dev.dev_no, 1);
    cdev_del(&led_dev.chrdev);
     
    return 0;
}

static struct platform_driver red_led_platform_driver = {
      .probe = led_red_driver_probe,
      .remove = led_red_driver_remove,
      .driver = {
        .name = "lhd,breathing_light_test",
        .owner = THIS_MODULE,
        .of_match_table = dts_match_table,         //通过设备树匹配
      },
};

module_platform_driver(red_led_platform_driver);

MODULE_AUTHOR("LHD");
MODULE_LICENSE("GPL");
```
将上述驱动编译为ko文件然后push进3588开发板里面

应用层程序

```c
#include "stdio.h"
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/ioctl.h>
#include <poll.h>
#include <stdint.h>

#define DEV_NAME   "/dev/breathing_light"

#define LED_PWM_CMD_SET_DUTY         0x01
#define LED_PWM_CMD_SET_PERIOD       0x02
#define LED_PWM_CMD_SET_BOTH         0x03
#define LED_PWM_CMD_ENABLE           0x04
#define LED_PWM_CMD_DISABLE          0x05

struct led_pwm_param {
    int duty_ns;
    int period_ns;
};

void sleep_ms(unsigned int ms)
{
    struct timeval delay;
	delay.tv_sec = 0;
	delay.tv_usec = ms * 1000; 
	select(0, NULL, NULL, NULL, &delay);
}

int main(int argc, char **argv)
{
    int fd;
    int ret;
  
	/* 2. 打开文件 */
	fd = open(DEV_NAME, O_RDWR | O_NONBLOCK);   // | O_NONBLOCK

	if (fd < 0)
	{
		printf("can not open file %s, %d\n", DEV_NAME, fd);
		return -1;
	}
     
    	int buf = 3;
	struct led_pwm_param led_pwm;
	
	led_pwm.duty_ns = 500;
	led_pwm.period_ns = 5000;
    	write(fd, &led_pwm, sizeof(led_pwm));
   	 sleep_ms(3000);

	while(1)
	{
		if(led_pwm.duty_ns<=500)
		{
			while(led_pwm.duty_ns<led_pwm.period_ns)
			{
				ioctl(fd, LED_PWM_CMD_SET_DUTY, &led_pwm.duty_ns);
				sleep_ms(50);
				led_pwm.duty_ns += 300;
			}
		}
		else
		{
			while(led_pwm.duty_ns > 500)
			{
				ioctl(fd, LED_PWM_CMD_SET_DUTY, &led_pwm.duty_ns);
				sleep_ms(50);
				led_pwm.duty_ns -= 300;
			}
		}
		
	}
	close(fd);
    
    return 0;
}
```
使用3588自带的编译器将用户程序编译进开发板

> prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-gcc apptest_breathing_light_.c -o testpwm
> adb push path/testpwm /userdata
> chmod 777 testpwm
> ./testpwm

最后可以看到灯明灭交替的效果

[video(video-vSITJV8u-1685090996829)(type-csdn)(url-https://live.csdn.net/v/embed/299757)(image-https://video-community.csdnimg.cn/vod-84deb4/b0de9630fba071ed9d426632b68f0102/snapshots/45e2d1e179874df9917cabcfb522b381-00001.jpg?auth_key=4838690316-0-0-83fb00e408ef39b4bf91669dbcd5d88e)(title-54cf1ab18dd620869c59ffc20)]

## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)