# 1. RKLLM 简介
## 1.1 RKLLM 工具链介绍
### 1.1.1 RKLLM-Toolkit 功能介绍

RKLLM-Toolkit 是为用户提供在计算机上进行大语言模型的量化、转换的开发套件。通过该
工具提供的 Python 接口可以便捷地完成以下功能：

1. 模型转换：支持将 Hugging Face 格式的大语言模型（Large Language Model, LLM）转换为
RKLLM 模型，目前支持的模型包括 LLaMA、Qwen/Qwen2、Phi2 等，转换后的 RKLLM 模型能
够在 Rockchip NPU 平台上加载使用。
2. 量化功能：支持将浮点模型量化为定点模型，目前支持的量化类型包括 w4a16 和 w8a8。

1.1.2 RKLLM Runtime 功能介绍

RKLLM Runtime 主 要 负 责 加 载 RKLLM-Toolkit 转换得到的 RKLLM 模型，并在
RK3576/RK3588 板端通过调用 NPU 驱动在 Rockchip NPU 上实现 RKLLM 模型的推理。在推理
RKLLM 模型时，用户可以自行定义 RKLLM 模型的推理参数设置，定义不同的文本生成方式，
并通过预先定义的回调函数不断获得模型的推理结果。

## 1.2 RKLLM 开发流程介绍

RKLLM 的整体开发步骤主要分为 2 个部分：模型转换和板端部署运行。

1. 模型转换：
在这一阶段，用户提供的 Hugging Face 格式的大语言模型将会被转换为 RKLLM 格式，
以便在 Rockchip NPU 平台上进行高效的推理。这一步骤包括：
a. 获取原始模型：获取 Hugging Face 格式的大语言模型；或是自行训练得到的大语言模
型，要求模型保存的结构与 Hugging Face 平台上的模型结构一致。
b. 模型加载：通过 rkllm.load_huggingface()函数加载原始模型。
c. 模型量化配置：通过 rkllm.build() 函数构建 RKLLM 模型，在构建过程中可选择是否
进行模型量化来提高模型部署在硬件上的性能，以及选择不同的优化等级和量化类型。
d. 模型导出：通过 rkllm.export_rkllm() 函数将 RKLLM 模型导出为一个.rkllm 格式文件，
用于后续的部署。
2. 板端部署运行：
这个阶段涵盖了模型的实际部署和运行。它通常包括以下步骤：
a. 模型初始化：加载 RKLLM 模型到 Rockchip NPU 平台，进行相应的模型参数设置来
定义所需的文本生成方式，并提前定义用于接受实时推理结果的回调函数，进行推理前准备。
b. 模型推理：执行推理操作，将输入数据传递给模型并运行模型推理，用户可以通过预
先定义的回调函数不断获取推理结果。
c. 模型释放：在完成推理流程后，释放模型资源，以便其他任务继续使用 NPU 的计算
资源。
这两个步骤构成了完整的 RKLLM 开发流程，确保大语言模型能够成功转换、调试，并最终
在 Rockchip NPU 上实现高效部署。

1.3 适用的硬件平台
本文档适用的硬件平台主要包括：RK3576、RK3588

# 2. 开发环境准备

在发布的 RKLLM 工具链压缩文件中，包含了 RKLLM-Toolkit 的 whl 安装包、RKLLM 
Runtime 库的相关文件以及参考示例代码，具体的文件夹结构如下：

doc
└──Rockchip_RKLLM_SDK_CN.pdf # RKLLM SDK 说明文档
rkllm-runtime
├──example
│ └── src
│ └── main.cpp
│ └── build-android.sh
│ └── build-linux.sh
│ └── CMakeLists.txt
│ └── Readme.md
├──runtime
│ └── Android
│ └── librkllm_api
│ └──arm64-v8a
│ └── librkllmrt.so # RKLLM Runtime 库
│ └──include
│ └── rkllm.h # Runtime 头文件
│ └── Linux
│ └── librkllm_api
│ └──aarch64
│ └── librkllmrt.so
│ └──include
│ └── rkllm.h
rkllm-toolkit
├──examples
│ └── huggingface
│ └── test.py
├──packages
│ └── md5sum.txt 
│ └── rkllm_toolkit-1.0.0-cp38-cp38-linux_x86_64.whl
rknpu-driver
└──rknpu_driver_0.9.6_20240322.tar.bz2

在本章中将会对 RKLLM-Toolkit 工具及 RKLLM Runtime 的安装进行详细的介绍，具体的使
用方法请参考第 3 章中的使用说明。

2.1 RKLLM-Toolkit 安装

本节主要说明如何通过 pip 方式来安装 RKLLM-Toolkit，用户可以参考以下的具体流程说明
完成 RKLLM-Toolkit 工具链的安装。

2.1.1 通过 pip 方式安装
2.1.1.1 安装 miniforge3 工具

为防止系统对多个不同版本的 Python 环境的需求，建议使用 miniforge3 管理 Python 环境。
检查是否安装 miniforge3 和 conda 版本信息，若已安装则可省略此小节步骤。
```
conda -V
# 提示 conda: command not found 则表示未安装 conda
# 提示 例如版本 conda 23.9.0
```

下载 miniforge3 安装包

```
wget -c https://mirrors.bfsu.edu.cn/github-release/condaforge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
```

安装 miniforge3

```
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
```

2.1.1.2 创建 RKLLM-Toolkit Conda 环境


进入 Conda base 环境
```
source ~/miniforge3/bin/activate # miniforge3 为安装目录
# (base) xxx@xxx-pc:~$
```
创建一个 Python3.8 版本（建议版本）名为 RKLLM-Toolkit 的 Conda 环境
```
conda create -n RKLLM-Toolkit python=3.8
```
进入 RKLLM-Toolkit Conda 环境
```
conda activate RKLLM-Toolkit
# (RKLLM-Toolkit) xxx@xxx-pc:~$
```

2.1.1.3 安装 RKLLM-Toolkit

在 RKLLM-Toolkit Conda 环境下使用 pip 工具直接安装所提供的工具链 whl 包，在安装过程
中，安装工具会自动下载 RKLLM-Toolkit 工具所需要的相关依赖包。
```
pip3 install rkllm_toolkit-1.0.0-cp38-cp38-linux_x86_64.whl
```

若执行以下命令没有报错，则安装成功。
```
python
from rkllm.api import RKLLM
```

2.2 RKLLM Runtime 库的使用
在所公开的的 RKLLM 工具链文件中，包括包含 RKLLM Runtime 的全部文件：
- lib/librkllmrt.so: 适用于 RK3576/RK3588 板端调用进行 RKLLM 模型部署推理的
RKLLM Runtime 库；
- include/rkllm_api.h: 与 librkllmrt.so 函数库相对应的头文件，其中包含相关结构体及
函数定义的说明；
在通过 RKLLM 工具链构建 RK3576/RK3588 板端的部署推理代码时，需要注意对以上头文
件及函数库的链接，从而保证编译的正确性。当代码在 RK3576/RK3588 板端实际运行的过程中，
同样需要确保以上函数库文件成功推送至板端，并通过以下环境变量设置完成函数库的声明：
```
ulimit -Sn 50000
export LD_LIBRARY_PATH=./lib
./llm_demo qwen.rkllm
```

2.3 RKLLM Runtime 的编译要求

在使用 RKLLM Runtime 的过程中，需要注意 gcc 编译器的版本问题。推荐使用交叉编译工具
gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu；具体的下载路径为：GCC_10.2 交叉编译工
具下载地址。请注意，交叉编译工具往往向下兼容而无法向上兼容，因此不要使用 10.2 以下的版
本。
若是选择使用 Android 平台，需要进行 Android 可执行文件的编译，推荐使用 Android NDK
工具进行交叉编译，下载路径为：Android_NDK 交叉编译工具下载地址，推荐使用 r18b 版本。
具体的编译方式也可以参考 RKLLM-Toolkit 工具链文件中的 example/build_demo.sh。
2.4 芯片内核更新
由于当前公开的固件内核驱动版本不支持 RKLLM 工具，因此需要更新内核。rknpu 驱动包支持两
个主要内核版本：kernel-5.10 和 kernel-6.1。对于 kernel-5.10，建议使用具体版本号 5.10.198，repo：
GitHub - rockchip-linux/kernel at develop-5.10；对于 kernel-6.1，建议使用具体版本号 6.1.57。可在
内核根目录下的 Makefile 中确认具体版本号。
更新步骤如下：
a. 下载压缩包 rknpu_driver_0.9.6_20240322.tar.bz2。
b. 解压该压缩包，将其中的 rknpu 驱动代码覆盖到当前内核代码目录。
c. 重新编译内核。
d. 将新编译的内核烧录到设备中。

## 使用环境

开发板：ArmSoM-Sige7
代码仓库：[rknn-llm](https://github.com/ArmSoM/rknn-llm)