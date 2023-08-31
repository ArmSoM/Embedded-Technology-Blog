# 前言
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)

- 本文主要讲解如何关于RK3588开发板UART的使用和调试方法，包括UART作为普通串口和控制台两种不同使用场景
# 一. 功能特点
Rockchip UART (Universal Asynchronous Receiver/Transmitter) 基于16550A串口标准，完整模块支持以下功能：

 - 支持5、6、7、8 bits数据位。
 -  支持1、1.5、2 bits停止位。 
 - 支持奇校验和偶校验，不支持mark校验和space校验。  
 - 支持接收FIFO和发送FIFO，一般为32字节或者64字节。 
 - 支持最高4M波特率，实际支持波特率需要芯片时钟分频策略配合。      
 - 支持中断传输模式和DMA传输模式。 支持硬件自动流控，RTS+CTS。

# 二、代码位置

在Linux kernel 中，使用8250串口通用驱动，以下为主要驱动文件：

> drivers/tty/serial/8250/8250_core.c # 8250串口驱动核心
> drivers/tty/serial/8250/8250_dw.c # Synopsis DesignWare 8250串口驱动
> drivers/tty/serial/8250/8250_dma.c # 8250串口DMA驱动
> drivers/tty/serial/8250/8250_port.c # 8250串口端口操作
> drivers/tty/serial/8250/8250_early.c # 8250串口early console驱动

SDK中提供的UART默认配置已经使用了8250驱动我们就不需要修改

# 三、硬件原理图
![调试串口](https://img-blog.csdnimg.cn/cd2810a248444f3297294ac1268e4771.png)
串口功能的硬件上比较简单，这是只附上调试串口的原理图
# 四、设备树配置
rk平台的设备树修改路径都是在kernel\arch\arm64\boot\dts\rockchip下面，具体哪个文件根据对应开发板来决定，通常描述设备硬件配置在rkxxxx.dtsi中，比如在rk3588s.dtsi中：

```
uart2: serial@feb50000 {
		compatible = "rockchip,rk3588-uart", "snps,dw-apb-uart";
		reg = <0x0 0xfeb50000 0x0 0x100>;
		interrupts = <GIC_SPI 333 IRQ_TYPE_LEVEL_HIGH>;
		clocks = <&cru SCLK_UART2>, <&cru PCLK_UART2>;
		clock-names = "baudclk", "apb_pclk";
		reg-shift = <2>;
		reg-io-width = <4>;
		dmas = <&dmac0 10>, <&dmac0 11>;
		pinctrl-names = "default";
		pinctrl-0 = <&uart2m1_xfer>;
		status = "disabled";
	};
```

## 4.1作为普通串口

![在这里插入图片描述](https://img-blog.csdnimg.cn/a966a2d0be374c27b14be217249ba4d1.png#pic_center)

假入我们想使用w3开发板上40PIN上的uart7
我们在dts可以使用如下配置打开

```
&uart7 {

status = "okay";

pinctrl-names = "default";

pinctrl-0 = <&uart7m1_xfer>;

};
```

## 4.2作为调试串口
Rockchip UART作为控制台，使用fiq_debugger流程。
在dts中fiq_debugger节点配置如下。由于fiq_debugger和普通串口互斥，在使能fiq_debugger节点后必须禁用对应的普通串口uart节点。

```
chosen: chosen {
	bootargs = "earlycon=uart8250,mmio32,0xfe660000 console=ttyFIQ0";
};
fiq-debugger {
	compatible = "rockchip,fiq-debugger";
	rockchip,serial-id = <2>;
	rockchip,wake-irq = <0>;
	/* If enable uart uses irq instead of fiq */
	rockchip,irq-mode-enable = <1>;
	rockchip,baudrate = <1500000>; /* Only 115200 and 1500000 */
	interrupts = <GIC_SPI 252 IRQ_TYPE_LEVEL_LOW>;
	pinctrl-names = "default";
	pinctrl-0 = <&uart2m0_xfer>;
	status = "okay";
};
&uart2 {
	status = "disabled";
};
```

 - rockchip,serial-id：使用的UART编号。修改serial-id到不同UART，fiq_debugger设备也会注册成
 - ttyFIQ0设备。 rockchip,irq-mode-enable：配置为1使用irq中断，配置为0使用fiq中断。
 - interrupts：配置的辅助中断，保持默认即可。 
 - pinctrl-0：使用的串口引脚
 - rockchip,baudrate：波特率配置

# 五、串口相关问题

## 5.1设备注册
普通串口设备将会根据dts中的aliase来对串口进行编号，对应注册成ttySx设备。注册的节点为/dev/ttyS4，命名规则是通过dts中的aliases来的。

```
aliases {
serial0 = &uart0;
serial1 = &uart1;
serial2 = &uart2;
serial3 = &uart3;
}
```
对应uart0注册为ttyS0，uart0注册为ttyS1，如果需要把uart3注册成ttyS1，可以进行以下修改

```
serial1 = &uart3;  
serial3 = &uart1;
```

## 5.2控制台打印相关
Rockchip UART打印通常包括DDR阶段、Miniloader阶段、TF-A (Trusted Firmware-A)阶段、OP-TEE阶段、Uboot阶段和Kernel阶段，我们平时主要关注的是uboot阶段和kernel阶段的打印，在这两个阶段我们可以尝试关闭所有打印或切换所有打印到其他UART，RK平台默认的调试串口是uart2_m0这一组引脚，假如现在我将打印换成其他串口，可以尝试以下做法。

### 5.2.1DDR Loader修改方法
DDR Loader中关闭或切换打印，需要修改DDR Loader中的UART打印配置，修改文件rkbin/tools/ddrbin_param.txt中的以下参数：

> uart id= # UART控制器id，配置为0xf为关闭打印 
> uart iomux= # 复用的IOMUX引脚 uart
> baudrate= # 115200 or 1500000

修改完成后，使用以下命令重新生成ddr.bin固件。

> ./ddrbin_tool ddrbin_param.txt rk3588_ddr_lp4_2112MHz_lp5_2736MHz_v1.09.bin

### 5.2.2Uboot修改方法
Uboot中关闭打印，需要在menuconfig中，打开配CONFIG_DISABLE_CONSOLE，保存到.config文件
Uboot中切换打印，由传参机制决定，不需要进行额外修改。uboot解析传参机制相关代码在arch/arm/mach-rockchip/board.c的board_init_f_init_serial()函数中。

### 5.2.3kernel修改方法
去掉打印需要在menuconfig中，关闭配置CONFIG_SERIAL_8250_CONSOLE。

> Device Drivers ---> 
>Character devices ---> 
> 				Serial drivers ---> 
> [ ]Console on 8250/16550 and compatible serial port

在dts配置中找到类似以下内容，并去掉UART基地址和console相关配置参数

```
chosen: chosen {
		bootargs = "earlycon=uart8250,mmio32,0xfeb50000 console=ttyFIQ0 irqchip.gicv3_pseudo_nmi=0 root=PARTUUID=614e0000-0000 rw rootwait";
	};
```
将0xfeb50000 console=ttyFIQ0 去掉，然后找到fiq-debugger节点，修改serial-id为0xffffffff，去掉UART引脚复用相关配置。注意，需要保持fiqdebugger节点使能，保持fiq-debugger流程系统才能正常启动

```
fiq_debugger: fiq-debugger {
		compatible = "rockchip,fiq-debugger";
		rockchip,serial-id = <0xffffffff>;
		rockchip,wake-irq = <0>;
		/* If enable uart uses irq instead of fiq */
		rockchip,irq-mode-enable = <1>;
		rockchip,baudrate = <1500000>;  /* Only 115200 and 1500000 */
		interrupts = <GIC_SPI 423 IRQ_TYPE_LEVEL_LOW>;
		status = "okay";
	};
```
切换打印串口例如将Kernel打印从UART2切换到UART3，在dts配置中找到类似以下内容，将UART基地址由UART2改为UART3.

> bootargs = "earlycon=uart8250,mmio32,0xfe670000 console=ttyFIQ0";

0xfe670000是UART3基地址，然后找到fiq-debugger节点，修改serial-id为3，修改UART3引脚复用配置pinctrl-0 = <&uart3m0_xfer>。注意，同时需要将切换为打印串口的UART3作为普通串口的节点禁用。

# 六、串口测试

在开发板上跑一套应用程序，可以发送数据，可以接收数据，测试方法可以短接TX_RX

```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <termio.h>
#include <time.h>
#include <pthread.h>

int read_data(int fd, void *buf, int len);
int write_data(int fd, void *buf, int len);
int setup_port(int fd, int baud, int databits, int parity, int stopbits);
void print_usage(char *program_name);

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t data_ready = PTHREAD_COND_INITIALIZER;
int data_available = 0;

void *read_thread(void *arg) {
    int fd = *(int *)arg;
    char buffer[1024]; // 存储读取的数据

    while (1) {
        int bytes_read = read_data(fd, buffer, sizeof(buffer));
        if (bytes_read > 0) {
            printf("Read Thread: Read %d bytes: %s\n", bytes_read, buffer);
        } else {
            // 处理读取错误或设备关闭的情况
            break;
        }
    }
    
    pthread_exit(NULL);
}

void *write_thread(void *arg) {
    int fd = *(int *)arg;
	char input[1024]; // 存储用户输入的数据

    while (1) {
        printf("Enter data to write (or 'q' to quit): ");
        fgets(input, sizeof(input), stdin);

        if (strcmp(input, "q\n") == 0 || strcmp(input, "Q\n") == 0) {
            // 用户输入 'q' 或 'Q'，退出循环
            break;
        }

        int len = strlen(input);
        int bytes_written = write_data(fd, input, len);
        if (bytes_written > 0) {
            printf("Write Thread: Wrote %d bytes: %s\n", bytes_written, input);
        }
    }
    
    pthread_exit(NULL);
}

int main(int argc, char *argv[]) //./a.out /dev/ttyS4 115200 8 0 1
{
    int fd;
    int baud;
    int len;
    int count;
    int i;
    int databits;
    int stopbits;
    int parity;

    if (argc != 6) {
        print_usage(argv[0]);
        return 1;
    }
 
    baud = atoi(argv[2]);
    if ((baud < 0) || (baud > 921600)) {
        fprintf(stderr, "Invalid baudrate!\n");
        return 1;
    }
 
    databits = atoi(argv[3]);
    if ((databits < 5) || (databits > 8)) {
        fprintf(stderr, "Invalid databits!\n");
        return 1;
    }
 
    parity = atoi(argv[4]);
    if ((parity < 0) || (parity > 2)) {
        fprintf(stderr, "Invalid parity!\n");
        return 1;
    }
 
    stopbits = atoi(argv[5]);
    if ((stopbits < 1) || (stopbits > 2)) {
        fprintf(stderr, "Invalid stopbits!\n");
        return 1;
    }
 
 
    fd = open(argv[1], O_RDWR, 0);
    if (fd < 0) {
        fprintf(stderr, "open <%s> error %s\n", argv[1], strerror(errno));
        return 1;
    }
 
    if (setup_port(fd, baud, databits, parity, stopbits)) {
        fprintf(stderr, "setup_port error %s\n", strerror(errno));
        close(fd);
        return 1;
    }
	pthread_t read_tid, write_tid;
    int ret;

    // 创建读取线程
    ret = pthread_create(&read_tid, NULL, read_thread, &fd);
    if (ret != 0) {
        fprintf(stderr, "Failed to create read thread\n");
        return 1;
    }

    // 创建写入线程
    ret = pthread_create(&write_tid, NULL, write_thread, &fd);
    if (ret != 0) {
        fprintf(stderr, "Failed to create write thread\n");
        return 1;
    }

    // 等待读取线程和写入线程结束
    pthread_join(read_tid, NULL);
    pthread_join(write_tid, NULL);
	
    close(fd);
 
    return 0;
}

static int baudflag_arr[] = {
    B921600, B460800, B230400, B115200, B57600, B38400,
    B19200,  B9600,   B4800,   B2400,   B1800,  B1200,
    B600,    B300,    B150,    B110,    B75,    B50
};
static int speed_arr[] = {
    921600,  460800,  230400,  115200,  57600,  38400,
    19200,   9600,    4800,    2400,    1800,   1200,
    600,     300,     150,     110,     75,     50
};

int speed_to_flag(int speed)
{
    int i;
 
    for (i = 0;  i < sizeof(speed_arr)/sizeof(int);  i++) {
        if (speed == speed_arr[i]) {
            return baudflag_arr[i];
        }
    }
 
    fprintf(stderr, "Unsupported baudrate, use 9600 instead!\n");
    return B9600;
}

static struct termio oterm_attr;

int setup_port(int fd, int baud, int databits, int parity, int stopbits)
{
    struct termio term_attr;
 
    
    if (ioctl(fd, TCGETA, &term_attr) < 0) {
        return -1;
    }
 
    
    memcpy(&oterm_attr, &term_attr, sizeof(struct termio));
 
    term_attr.c_iflag &= ~(INLCR | IGNCR | ICRNL | ISTRIP);
    term_attr.c_oflag &= ~(OPOST | ONLCR | OCRNL);
    term_attr.c_lflag &= ~(ISIG | ECHO | ICANON | NOFLSH);
    term_attr.c_cflag &= ~CBAUD;
    term_attr.c_cflag |= CREAD | speed_to_flag(baud);
 
    
    term_attr.c_cflag &= ~(CSIZE);
    switch (databits) {
        case 5:
            term_attr.c_cflag |= CS5;
            break;
 
        case 6:
            term_attr.c_cflag |= CS6;
            break;
 
        case 7:
            term_attr.c_cflag |= CS7;
            break;
 
        case 8:
        default:
            term_attr.c_cflag |= CS8;
            break;
    }
 
    
    switch (parity) {
        case 1:  
            term_attr.c_cflag |= (PARENB | PARODD);
            break;
 
        case 2:  
            term_attr.c_cflag |= PARENB;
            term_attr.c_cflag &= ~(PARODD);
            break;
 
        case 0:  
        default:
            term_attr.c_cflag &= ~(PARENB);
            break;
    }
 
 
    
    switch (stopbits) {
        case 2:  
            term_attr.c_cflag |= CSTOPB;
            break;
 
        case 1:  
        default:
            term_attr.c_cflag &= ~CSTOPB;
            break;
    }
 
    term_attr.c_cc[VMIN] = 1;
    term_attr.c_cc[VTIME] = 0;
 
    if (ioctl(fd, TCSETAW, &term_attr) < 0) {
        return -1;
    }
 
    if (ioctl(fd, TCFLSH, 2) < 0) {
        return -1;
    }
 
    return 0;
}
 
 
int read_data(int fd, void *buf, int len)
{
    int count;
    int ret;
 
    ret = 0;
    count = 0;
 
    //while (len > 0) {
 
    ret = read(fd, (char*)buf + count, len);
    if (ret < 1) {
        fprintf(stderr, "Read error %s\n", strerror(errno));
        //break;
    }
 
    count += ret;
    len = len - ret;
 
    //}
 
    *((char*)buf + count) = 0;
    return count;
}
 
 
int write_data(int fd, void *buf, int len)
{
    int count;
    int ret;
 
    ret = 0;
    count = 0;
 
    while (len > 0) {
 
        ret = write(fd, (char*)buf + count, len);
        if (ret < 1) {
            fprintf(stderr, "Write error %s\n", strerror(errno));
            break;
        }
 
        count += ret;
        len = len - ret;
    }
 
    return count;
}

void print_usage(char *program_name)
{
    fprintf(stderr,
            "*************************************\n"
            "  A Simple Serial Port Test Utility\n"
            "*************************************\n\n"
            "Usage:\n  %s <device> <baud> <databits> <parity> <stopbits> \n"
            "       databits: 5, 6, 7, 8\n"
            "       parity: 0(None), 1(Odd), 2(Even)\n"
            "       stopbits: 1, 2\n"
            "Example:\n  %s /dev/ttyS4 115200 8 0 1\n\n",
            program_name, program_name
           );
}
```
运行效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ebd9dc29cbf344dfb71653d21dadc8ee.png#pic_center)
 ## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)