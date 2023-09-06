

# Introduction to PWM

- Series Contents
- PWM stands for Pulse Width Modulation. It is a technology used to control electronic devices by modifying the pulse width of electronic signals to regulate the devices.

## Basic Concepts   

A PWM signal consists of a periodic pulse train at a fixed frequency. The width (duration) of each pulse can be adjusted as needed. By varying the duty cycle of the pulses, the average voltage or current output can be controlled, which enables control of the device.

Three key terms for PWM.  

Frequency: The frequency of a PWM signal refers to the number of pulses per second.

Period: The period of a PWM signal is the time it takes to complete one pulse cycle. It is the inverse of frequency, measured in seconds. The period can be calculated by taking the inverse of frequency. For example, a 10kHz PWM signal has a period of 0.1ms (100 microseconds). 

Duty Cycle: The duty cycle refers to the ratio of pulse width to the period in a PWM signal. It indicates the fraction of time the pulse is active in one period, usually expressed as a percentage. 0% duty cycle means the pulse is inactive (continuously low), while 100% duty cycle means the pulse is active for the entire period (continuously high). In practice, duty cycle can be adjusted anywhere from 0% to 100% to achieve the desired control effect.

# PWM Driver 

The PWM driver is a generic driver that SoC vendors enable in their SDKs by default.

## Driver Files

The PWM driver files are located at:

> drivers/pwm/pwm-rockchip.c

The PWM driver is already loaded in SDKs by default . The below focus is how to use PWM. 

## DTS Node Configuration

Refer to DTS configuration documentation:

> Documentation/devicetree/bindings/pwm/pwm.txt 

Here is an example:

```
Node name {
  compatible = "Driver matching string";
  pwms = <&pwmX 0 25000 0>;  
};

&pwmX {
  status = "okay";
  pinctrl-names = "active";
  pinctrl-0 = <&pwmX_pin_pull_down>; 
};
```

The  PWMS parameters are:

1. Index - the per-chip index of the PWM to request, usually 0 since Rockchip has one PWM per chip. 

2. Period - the PWM output waveform period in ns. For example, 25000 means the desired PWM output period is 40kHz.

3. Polarity - optional, negative polarity in the example.

# How to use PWM

The PWM provides userspace interfaces under /sys/class/pwm/. After driver loading, the pwmchip0 directory is created under /sys/class/pwm/. 

Writing 0 to export enables PWM timer 0 and creates a pwm0 directory. Conversely, writing 0 to unexport disables timer 0 and removes pwm0. 

The pwm0 directory contains:

> enable - write 1 to enable, 0 to disable PWM.

> polarity - normal or inversed options which indicates a transmitter pin level inversion;

> duty_cycle : Within one period,High level duration in ns  for normal mode  or low level duration in ns  for reversed mode.

> period :PWM waveform period in ns.

Here is an example of pwmchip0, configuring pwm0 with 100kHz output frequency, 50% duty cycle, and normal polarity:

```
cd /sys/class/pwm/pwmchip0/
echo 0 > export
cd pwm0
echo 10000 > period  
echo 5000 > duty_cycle
echo normal > polarity
echo 1 > enable
```

## Application Example

PWM is commonly used in electronic devices like fan motor control, TV backlights, LED lighting, power tool motor control, car heaters etc.

Here is a simple example of using PWM to control an LED to achieve a breathing light effect. The breathing light requires integrating the drivers of LED  and the PWM , with data exchange between them. We can call one driver from within another driver. The LED is the requested device , which uses PWM driven by the default driver.

On the hardware, we identified the pin with PWM function on the development board:

![ArmSoM-W3_40pin](https://img-blog.csdnimg.cn/23eb9ffe364348a4aa3dc9b86d0770ea.png)

![pwm8](https://img-blog.csdnimg.cn/38a3fef1095842d7a359f43682cf7d3e.png)

The device tree is modified as: 

```
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

A driver is implemented using the PWM subsystem internally. This achieves a driver inside a driver.

## Example Code

Driver code

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

#define RED_LED_DTS_COMPATIBLE       "lhd,breathing_light_test"       /* Device tree node matching attributes */

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
    
	//pwm_set_periodnnn(led_dev.red_led_pwm, PWM_POLARITY_INVERSED);//Set the polarity of PWM signal
	pwm_enable(led_dev.red_led_pwm);//Enable specified PWM device to start outputting PWM signals

    printk("red_led_pwm open\r\n");
    return ret;
}

static ssize_t red_led_drv_write (struct file *file, const char __user *buf, size_t size, loff_t *offset)
{
    int err;

    if (size != sizeof(led_pwm)) return -EINVAL;

	err = copy_from_user(&led_pwm, buf, size);
    if (err > 0) return -EFAULT;

	pwm_config(led_dev.red_led_pwm, led_pwm.duty_ns, led_pwm.period_ns);//Configure basic parameters of PWM device, such as frequency, duty cycle, etc.
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

    pwm_config(led_dev.red_led_pwm, 0, 5000);//Configure basic parameters of PWM device, such as frequency, duty cycle, etc.
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

/*Matching list of device tree */
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
    child = of_get_next_child(tdev->of_node, NULL);      /* Obtain the device tree subnodes */
	if (!child) {
        return -EINVAL;
    }

    led_dev.red_led_pwm = devm_of_pwm_get(tdev, child, NULL);     /* Get the PWM devices from the subnodes, obtaining this device from the device tree is enough */
    if (IS_ERR(led_dev.red_led_pwm)) {
        printk(KERN_ERR"can't get breathing_light!!\n");
        return -EFAULT;
    }

    ret = alloc_chrdev_region(&led_dev.dev_no, 0, 1, "breathing_light");//Dynamically allocate major number for character device
	if (ret < 0) {
		pr_err("Error: failed to register mbochs_dev, err: %d\n", ret);
		return ret;
	}

	cdev_init(&led_dev.chrdev, &red_led_drv);//Initialize character device structure cdev

	cdev_add(&led_dev.chrdev, led_dev.dev_no, 1);// Add the initialized character device structure cdev into the system

    led_dev.led_class = class_create(THIS_MODULE, "breathing_light");// Create a device class and register it to the kernel
	err = PTR_ERR(led_dev.led_class);
	if (IS_ERR(led_dev.led_class)) {
        goto failed1;
	}

    tdev = device_create(led_dev.led_class , NULL, led_dev.dev_no, NULL, "breathing_light"); // Create a device instance and register it to the device class
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
        .of_match_table = dts_match_table,         //Match through device tree
      },
};

module_platform_driver(red_led_platform_driver);

MODULE_AUTHOR("LHD");
MODULE_LICENSE("GPL");
```

Compile the above driver into a ko file and push it into the 3588 development board.

Application code

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
  
	/* 2. Open the file */
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

Use the built-in compiler in RK3588 to compile the user program into the development board.

> prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-gcc apptest_breathing_light_.c -o testpwm
> adb push path/testpwm /userdata
> chmod 777 testpwm
> ./testpwm

Finally, the alternating dimming and brightening effect of the light can be observed.

[video(video-vSITJV8u-1685090996829)(type-csdn)(url-https://live.csdn.net/v/embed/299757)(image-https://video-community.csdnimg.cn/vod-84deb4/b0de9630fba071ed9d426632b68f0102/snapshots/45e2d1e179874df9917cabcfb522b381-00001.jpg?auth_key=4838690316-0-0-83fb00e408ef39b4bf91669dbcd5d88e)(title-54cf1ab18dd620869c59ffc20)]