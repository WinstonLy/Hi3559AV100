## 一 概述

### 1.1 SVP简介

SVP是海思媒体处理芯片只能视觉异构加速平台。该平台包含了CPU、DSP、NNIE等多个硬件处理单元和运行在这些硬件上SDK开发环境，以及配套的工具链开发环境。

### 1.2 SVP开发框架

![](D:\workspace\Hi3559AV100\image\SVP.png)



Hi3559AV100使用硬件资源

- CPU：双核A73 + 双核A53
- DSP：4个
- NNIE：2个



## 二 NNIE开发指南

### 2.1 NNIE介绍

NNIE是Neural Network Inference Engine的简称，是专门针对神经网络进行加速处理的硬件单元，支持现有的大部分的公开网络（分类网络，检测网络，场景分割网络）。目前NNIE配套软件和工具链只支持Caffe框架。

### 2.2 重要概念

- 网络分段

    对于NNIE不支持的某些网络层节点， 编译器支持用户对网络分段，不执行的部分编译器不去编译，由用户自己用CPU去实现。建议用户尽量使用NNIE支持的层去实现网络模型；NNIE不支持的段数越多，网络切分越碎，软硬件交互频繁，效率越低。

- 句柄（handle）

    用户在调用NNIE API创建任务时，系统会为每一个任务分配一份handle，用于标识不同的任务。

- 及时返回结果标志bInstant

    用户在创建某个任务后，希望及时得到该完成的信息则需要时，将bInstant设置为 HI_TRUE。否则，如果用户不关心该任务是完成建议将bInstant设置为 设置为HI_FALSE，这样可以与后续任务组链执行，减少中断次数，提升性能。

- 查询

    用户根据返回的handle，调用HI_MPI_SVP_NNIE_Query可以查询对应算子任务是否完成。

- 及时刷cache

    NNIE硬件只能从DDR中获取数据。为保证输入输出数据不被CPU cache干扰，调用相关接口将数据从cache刷到DDR,以供IVE使用。

- 跨度（stride）

    一行的有效数据byte数目 + 为硬件快速跨越到下一行补齐的一些无效byte数目。

- 对齐

    硬件为了快速访问内存首地址或者跨行访问数据，要求内存地址或内存跨度必须为对齐系数的倍数。

### 2.3 开发流程

以 Caffe 框架上训练的模型为例,NNIE 的开发流程如图 3-1 所示。在 Caffe 上训练、使用 NNIE 的 mapper 工具转化都是离线的。通过设置不同的模式,mapper 将*.caffemodel 转化成在仿真器、仿真库或板端上可加载执行的数据指令文件。一般在开发前期,用户可使用仿真器对训练出来的模型进行精度、性能、带宽进行初步评估,符合用户预期后再使用仿真库进行完整功能的仿真,最后将程序移植到板端。

### 2.4 网络层的划分

标准层：NNIE 支持的 Caffe 标准层,比如 Convolution,Pooling 层等;

扩展层：NNIE 支持的公开但非 Caffe 标准层,分为 2 种:

- 一种是基于 Caffe 框架进行自定义扩展的层,比如 Faster RCNN 中的ROIPooling层、SSD中Normalize层、RFCN中的PSROIPooling 层,SegNet 中的 UpSample 层等;
- 另外一种是来源于其他深度学习框架的自定义层,比如 YOLOv2 中 Passthrough层等;

Non-support 层：NNIE不支持的层,比如Caffe中专用于Tranning的层、非Caffe框架中的一些层或用户自定义的私层等。



### 2.5 扩展层规则

Faster RCNN、SSD、RFCN 和 SegNet 等网络都包含了一些原始 Caffe 中没定义的层结构,如 ROIPooling、Normalize、PSROI Pooling 和 Upsample 等。NNIE 的 mapper 目前仅支持 Caffe 框架,且以 Caffe-1.0 为基础。为了使 mapper 能支持这些网络,需要对原始的 Caffe 进行扩展。

#### 2.5.1 扩展 Caffe proto 文件

扩展层可能涉及到一些相关的输入参数,因此需要扩展 Caffe 的 proto 文件,使得Caffe在解析 prototxt 文件时能够识别出对应的参数声明。

#### 2.5.2 扩展层代码实现

经过上述 caffe.proto 文件的扩展之后,Caffe 就能够识别 prototxt 中层参数相关的设定。但若需要支持扩展层的解析、training、inference,还需要根据扩展层的处理方式,完成对应底层代码实现,如果该层处理要在 CPU 上运行,就要实现对应的 C++代码,如果该层处理要在 GPU 上运行,就要实现对应的 CUDA C/C++代码。

#### 2.5.3 扩展层在 prototxt 中的定义

扩展层的实现可以在 Caffe 或其他深度学习框架中实现,扩展层中的第一种是基于Caffe 框架进行扩展实现的,如 ROI Pooling、Normalize、PSROI Polling 和 Upsample 层等,因此公开的 prototxt 标准定义。而扩展层中的第二种并不是在Caffe 框架下实现的,对于这类网络中的自定义层,也需要给出 prototxt 的标准定义,用于在 prototxt 中定义网络中相应的层。

### 2.6 Non-support 层处理方式

当网络中存在 Non-support 层时,需要将网络进行切分,不支持的部分由用户使用CPU或者DSP等方式实现,统称为非 NNIE 方式。由此整个网络会出现 NNIE->非 NNIE-

nie_mapper 将 NNIE 的 Non-support 层分为两种,“Proposal”层和“Custom”层:

- Proposal 层输出的是矩形信息(Bbox,即 Bounding box)
- Custom 层输出的是 tensor(feature map、vector)

## 三 RuyiStudio 工具使用指南

RuyiStudio 集成 windows版的NNIE mapper 和仿真库,具有生成NNIE wk功能、仿真NNIE功能,同时具有代码编辑、编译、调试、执行功能、网络拓扑显示、目标检测画框、向量相似度对比、调试定位信息获取等功能。
RuyiStudio 集成 windows 版的 NNIE mapper 基于 Visual Studio 2015 64bit 版本编译,所以其依赖库也需要使用 Visual Studio 2015 64 bit 进行编译。
RuyiStudio 集成仿真库基于 MinGW-W64 7.3.0 编译,所以其依赖的编译链环境也需要是 MinGW-W64

### 3.1 RuyStudio安装

按照官方教程利用相关脚本直接安装，安装失败的包可以直接到相应网站下载下来到具体目录下安装。如opencv_python-3.4.0.12-cp35-cp35m-win_amd64.whl安装失败，下载相应文件到python35\Lib\site-packages，用cmd进入到该目录下，用pip install <软件名>指令进行安装。

### 3.2 工程文件说明

- *.cfg：用于配置NNIE mapper运行参数的cfg文件；
- mapper文件夹用于存放NNIE mapper后输出的文件；
- temp文件夹用于存放mapper前准备过程中的输出文件；
- sim_out保存调用仿真库运行时根据hisilicon/nnie_sim_ini配置的仿真输出文件。