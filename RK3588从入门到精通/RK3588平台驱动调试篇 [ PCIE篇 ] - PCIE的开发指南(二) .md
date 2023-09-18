RK3588平台驱动调试篇 [ PCIE篇 ] - PCIE的开发指南(二) 

# 一、前言

上一篇已经介绍过如何在3588上使用pcie的资源，这一篇介绍在Linux系统下如何应用pcie接上的设备

# 二、 PCI 配置空间

一个PCIe系统最多有256条Bus，每条Bus上最多挂32个Device，每个Device最多又能实现8个Function，每个Function对应着4KB的配置空间。PCI设备拥有256B的配置空间，PCIe还提供另外4KB的扩展，这256B的配置空间中前64B是规范了的，其他的字节是各个厂商自己定义的。

## 2.1 PCI 设备的地址组成

 PCI设备的地址是由三个部分组成的，通常以"域(Domain):总线(Bus):设备(Device).功能(Function)"的形式表示：

- 域（Domain）： 域是PCI设备的最高级别的地址组成部分。它用于标识不同的PCI总线。通常，大多数系统只有一个域，因此它的值为0。但在某些情况下，多个PCI域可以用于连接不同的PCI总线，每个域都有唯一的编号。

- 总线（Bus）： 总线标识PCI设备连接到计算机主板上的不同PCI总线。每个总线可以连接多个PCI设备。总线号通常是一个介于0和255之间的整数。

- 设备（Device）： 设备标识特定总线上的不同PCI设备。每个PCI总线可以连接多个设备，每个设备都有唯一的设备号，通常是0到31之间的整数。

- 功能（Function）： 功能标识PCI设备中的不同功能单元。有些PCI设备具有多个功能，每个功能都有唯一的功能号，通常是0到7之间的整数。大多数PCI设备只有一个功能。


这个地址组成使得系统能够唯一地标识和管理各种PCI设备，以便它们可以有效地与计算机系统进行通信。在使用工具如lspci时，这些地址通常用于显示和识别PCI设备。
将上一篇介绍的ArmSom-W3开发板的M.2插槽接好对应模组，上电后使用lspci命令查看：

```
root@linaro-alip:/home/linaro# lspci
0000:00:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd Device 3588 (rev 01)
0000:01:00.0 Non-Volatile memory controller: Intel Corporation NVMe Optane Memory Series
0002:20:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd Device 3588 (rev 01)
0002:21:00.0 Network controller: Realtek Semiconductor Co., Ltd. Device b852
0004:40:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd Device 3588 (rev 01)
0004:41:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
```

## 2.2 设备地址分析

lspci命令的输出会列出所有PCI设备的信息，包括设备的制造商、型号、PCI地址等。输出通常以文本形式提供，并按总线地址（BDF：Bus, Device, Function）的顺序排列。

**上述命令使用结果分析：**

1. 
   0000:00:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd Device 3588 (rev 01)

   设备地址：0000:00:00.0
   设备类型：PCI桥接器（PCI Bridge）
   制造商：Fuzhou Rockchip Electronics Co., Ltd
   设备型号：Device 3588
   设备版本：rev 01
   此设备是一种PCI桥接器，通常用于将其他PCI设备连接到计算机主板上。

2. 0000:01:00.0 Non-Volatile memory controller: Intel Corporation NVMe Optane Memory Series

   设备地址：0000:01:00.0
   设备类型：非易失性内存控制器（Non-Volatile Memory Controller）
   制造商：Intel Corporation
   设备型号：NVMe Optane Memory Series
   此设备是Intel Corporation生产的非易失性内存（NVMe）控制器，通常用于管理NVMe存储设备，如高速固态硬盘（SSD）。

3. 0002:21:00.0 Network controller: Realtek Semiconductor Co., Ltd. Device b852

   设备地址：0002:21:00.0
   设备类型：网络控制器（Network Controller）
   制造商：Realtek Semiconductor Co., Ltd.
   设备型号：Device b852
   此设备是一块Realtek Semiconductor Co., Ltd生产的网络控制器，通常用于连接计算机到网络。

4. 0004:41:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)

   设备地址：0004:41:00.0
   设备类型：以太网控制器（Ethernet Controller）
   制造商：Realtek Semiconductor Co., Ltd.
   设备型号：RTL8125 2.5GbE Controller
   设备版本：rev 05
   此设备是一块Realtek Semiconductor Co., Ltd生产的以太网控制器，支持2.5千兆比特每秒（2.5GbE）的网络连接速度，用于连接计算机到网络。

设备地址"0000:01:00.0"表示了一个PCI设备在系统中的唯一标识。这个地址可以被分解为以下部分来进行分析：

> 域（Domain）： 在这种情况下，域的值为"0000"，通常情况下，大多数系统只有一个域，所以它的值通常是"0000"。
>
> 总线（Bus）： 总线的值为"01"，表示这个PCI设备连接到系统的第1个PCI总线。每个总线可以连接多个PCI设备。
>
> 设备（Device）： 设备的值为"00"，表示在该总线上的第1个PCI设备。每个总线可以连接多个设备，它们分别具有唯一的设备号。
>
> 功能（Function）： 功能的值为"0"，表示这个PCI设备只有一个功能单元。一些PCI设备具有多个功能单元，每个功能单元都有唯一的功能号。
>
> 这个地址用于唯一标识PCI设备，以便系统可以识别和管理它们。您可以使用这个地址来查询或配置PCI设备，以及了解它们在系统中的物理位置和特征。
>

# 三、PCI设备使用

pcie接口接高速固态硬盘（SSD）的情景较多，这里使用由Intel Corporation生产的非易失性内存（NVMe）控制器，ArmSom-W3开发板使用的内核已经确保系统上已经加载了相应的NVMe驱动程序，并且操作系统能够正确识别和管理NVMe设备。

## 3.1 NVMe控制器使用

**这里介绍一下使用NVMe控制器的基本步骤：**

1. 检查NVMe设备是否被识别：
    运行以下命令，查看系统是否正确识别了NVMe设备

   ```
   root@linaro-alip:/home/linaro# lspci | grep NVMe
   0000:01:00.0 Non-Volatile memory controller: Intel Corporation NVMe Optane Memory Series
   
   ```

   如果您看到与Intel Corporation相关的NVMe设备信息，则表示设备已经被识别。

2. 检查NVMe驱动程序是否加载
   使用以下命令检查系统是否已加载了NVMe驱动程序：

   ```
   lsmod | grep nvme
   ```

   如果输出中显示了与NVMe驱动程序相关的模块（例如nvme），则表示驱动程序已加载。

3. 查看NVMe设备信息： 
   使用以下命令查看NVMe设备的详细信息，包括设备的名称、容量等：

   ```
   root@linaro-alip:/home/linaro# nvme list
   Node             SN                   Model                                    Namespace Usage                      Format           FW Rev  
   ---------------- -------------------- ---------------------------------------- --------- -------------------------- ---------------- --------
   /dev/nvme0n1     PHBT8506028Z016N     INTEL MEMPEK1J016GAL                     1          14.40  GB /  14.40  GB    512   B +  0 B   K4110420
   ```


   或者使用以下命令查看设备的分区信息：

   ```
   root@linaro-alip:/home/linaro# lsblk
   NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   mmcblk0      179:0    0 29.1G  0 disk 
   ├─mmcblk0p1  179:1    0    4M  0 part 
   ├─mmcblk0p2  179:2    0    4M  0 part 
   ├─mmcblk0p3  179:3    0   64M  0 part 
   ├─mmcblk0p4  179:4    0  128M  0 part 
   ├─mmcblk0p5  179:5    0   32M  0 part 
   ├─mmcblk0p6  179:6    0   14G  0 part /
   ├─mmcblk0p7  179:7    0  128M  0 part /oem
   └─mmcblk0p8  179:8    0 14.8G  0 part /userdata
   mmcblk0boot0 179:32   0    4M  1 disk 
   mmcblk0boot1 179:64   0    4M  1 disk 
   nvme0n1      259:0    0 13.4G  0 disk 
   ```

   在输出中，NVMe设备通常以/dev/nvmeXnY的形式表示，其中X是NVMe设备的编号，Y是分区编号。

注意：
 ArmSom-W3固件里NVMe驱动程序相关的模块已经加载至内核里面
 Linux系统通常使用`nvme-cli`工具执行各种操作，如查看设备信息、执行固件更新、执行健康检查等

## 3.2 挂载设备

NVMe设备是/dev/nvme0n1，总容量为14.40 GB，当前使用了14.40 GB

**使用以下命令挂载它：**

```
root@linaro-alip:/dev# mount /dev/nvme0n1 /mnt
[ 4399.143769] EXT4-fs (nvme0n1): recovery complete
[ 4399.145058] EXT4-fs (nvme0n1): mounted filesystem with ordered data mode. Opts: (null)
```

recovery complete：这是文件系统（EXT4）的恢复消息，它表明文件系统在挂载前进行了一次恢复操作，以确保文件系统的一致性。

mounted filesystem with ordered data mode. Opts: (null)：这是文件系统挂载成功的消息，表明文件系统已经成功挂载，并且使用了"ordered data mode"模式。括号中的"(null)"表示没有指定特定的挂载选项。

**使用以下命令卸载设备：**

```
umount /mnt
```

对于存储设备，还可以进行分区和格式化操作，这个看个人需要，可以使用工具如`fdisk`或`parted`来创建分区，并使用mkfs命令格式化分区

## 3.3 读写测试

对NVMe设备进行读写测试，可以使用一些专门的基准测试工具，例如fio或dd命令。
下面是一些基本的操作步骤：

1. 使用fio进行读写测试：

   安装fio工具

   ```
   apt-get install fio
   ```

   创建一个fio测试配置文件，创建一个名为test.fio的文件，内容如下：

   ```
   [sequential-read]
   filename=/dev/nvme0n1
   rw=read
   bs=4k
   size=1G
   ```

   这个配置文件将对NVMe设备执行4KB块大小的1GB顺序读取测试。可以根据需要调整参数。
   

2. 使用dd命令进行读写测试：

   运行以下写测试命令：

   ```
   sudo dd if=/dev/zero of=/dev/nvme0n1 bs=1M count=1000
   ```

   其中if参数是输入文件（通常是/dev/zero，用于写入测试），of参数是输出文件（通常是您的NVMe设备），bs参数是块大小，count参数是要执行的块数

   运行以下读测试命令：

   ```
   sudo dd if=/dev/nvme0n1 of=/dev/null bs=1M count=1000
   ```

   

读写性能可能会受到多种因素的影响，包括设备型号、硬件配置和测试条件等