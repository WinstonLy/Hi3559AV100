### 一．MIPI使用指南

#### 1.1        概述

MIPI Rx通过低电压差分信号接收原始视频数据，将接收到的串行差分信号（serial differential signal）转化为DC（ Digital Camera）时序后传递给下一级模块VICAP（ Video Capture ）

MIPI Rx支持MIPI D-PHY、LVDS （ Low-Voltage Differential Signal）、HiSPi （High-Speed Serial Pixel Interface）等串行视频信号输入，同时兼容DC视频接口。

#### 1.2        重要概念

\- MIPI

MIPI的全称是Mobile Industry Processor Interface（移动行业处理器接口），开发板描述的MIPI接口特指物理层使用D-PHY传输规范，协议层使用CSI-2的通信接口。

\- LVDS

LVDS的全称是Low Voltage differential Signaling（低压差分信号），通过同步码区分消隐区和有效数据。

\- SLVS-EC

SLVS-EC的全称是Scalable Low Voltage Signaling Embedded Clock，是与MIPI并列的接口，用于用于高帧率和高分辨率图像采集。

\- Lane

用于连接发送端和接收端的一对高速差分线，既可以是时钟Lane，也可以是数据Lane。

 

\- Link

发送端和接收端之间的时钟Lane和至少一个数据Lane组成一个Link，本文中的link是一个软件概念，每一个link包括两个数据lane。

\- 同步码

MIPI接口使用CSI-2里面的短包进行同步，LVDS使用同步码区分有效数据和消隐区。

\- LVDS有两种同步方式：

​    一种使用SOF/EOF 表示帧起始和结束，使用SOL/EOL表示行的起始和结束。另一种使用SAV（invalid）EAV（invalid）表示消隐区的无效数据开始和结束，使用SAV（valid）EAV（valid）表示有效像素数据的开始和结束。  

 

##### 1.3        功能描述

 

\- MIPIRx是一个支持多种差分视频输入接口的采集单元，通过combo-PHY接收MIPI/LVDS/sub-LVDS/HiSP/DC接口的数据，通过不同的功能模式配置，MIPI Rx可以支持多种速度和分辨率的数据传输需求，支持多种外部输入设备。

\- SLVS-EC接口支持更高帧率更大分辨率图像的采集，通过SLVS-EC的PHY接收高速串行的数据转化为DC（Digital Camera）时序，通过不同的功能模式配置，SLVS-EC可以支持多种速度和分辨率的数据传输需求，支持多种外部输入设备

### 二．  存储器相关说明

#### 2.1Hi3559AV100存储器参数表

​       存储器                数据位宽       频率                 容量

 

SPI NAND FLASH     1/2/4bit      单沿100MHz       2Gbit                                                                  

​										             双沿100MHz                            

​       eMMC                    8bit           200MHz            8GB

​       DDR                       64bit       1333MHz          单颗8Gb 总32Gb

 

#### 2.2DDR内存管理说明

\- 所有DDR内存中，一部分由操作系统管理，称为OS内存；另一部分由osal模块管理，供媒体业务单独使用，称为MMZ内存。

\- Single端的OS内存起始地址为04x200000，内存大小可通过bootargs进行配置，启动参数设置setenv bootargs 'mem=256M .. '，表示分配给Single OS操作系统内存为256M，可以根据实际情况进行调整。

\- MMZ内存由osal 内核模块管理（mpp/ko 目录下的hi_osal.ko），加载osal模块时，通过模块参数指定其起始地址及大小，可在load 脚本中修改MMZ的起始地址mmz_ start 及大小mmz_ size。每个MMZ分为许多区域（zone），可以在mpp/ko目录下的load脚本下设定，如有特殊需求可以单独设立分区，每个分区有一个name。每个区域又划分为多个mmb。

\- 请注意任何区域的内存划分都不能重叠。

 

#### 2.3DDR内存分配说明

​       Hi3559AV100的DDR地址空间是从0x40000000开始的。

​       Hi3559AV100的DDR内存空间主要分为以下几个部分：

\- 核间通信共享内存（A53MP+A73MP，A53UP，cortex-M7，DSP间）

\- DSP0/1/2/3 HuaWei LiteOS系统内存

\- A53UP Huawei LiteOS系统内存

\- A53MP+A73MP使用的MMZ区域

\- A53UP使用的MMZ区域。

   可以在osdrv/osdrv_mem_cfg.sh 文件中配置内存分配，然后编译整|个osdrv。

#### 2.4DDR内管理说明示意

 

 

DDR:                

--------|-------|  0x40000000   # Memory managed by OS.           

64M-- | OS  |                                                                          

--------|--------|  0x44000000   # Memory managed by MMZ block anonymous.         

442M | MMZ|                                                                          

--------|--------|  0x5FA00000   # Memory managed by MMZ block jpeg.

6M  | jpeg  |     

--------|--------|  0x60000000   # End of memory managed by MMZ.

 

 

 

### 三，  视频缓存池

 

#### 3.1概念

（1）视频缓存池(VB，video buffer)，是一段用于暂存视频数据、进行运算的内存。

 

（2）视频的裁剪、缩放、修正处理等各种操作，本质上是对内存中的数据进行运算。

 

（3）视频缓存池的内存由MPP来维护。

 

\- 系统启动时，把整个DDR分成2部分：系统部分（由linux kernel来维护管理）和mpp部分（由mpp系统来维护管理）

 

（4）缓存池的数量，缓存块的数目和大小，可以由用户程序设置好参数，调用MPP的相应API来向MPP申请分配。

 

（5）一组大小相同、物理地址连续的缓存块组成一个视频缓存池。必须在系统初始化之前配置公共视频缓存池。根据业务的不同，公共缓存池的数量、缓存块的大小和数量不同。

 

#### 3.2相关API接口说明

 

​       视频缓存池大小计算接口                     接口简介

 

​       COMMON_ GetPicBufferConfig    一般linear格式的YUV各部分数据大小

 

​       COMMON_ GetPicBufferSize      一般linear格式的YUV缓存池

 

​       VI_ GetRawBufferSize          VI写出的Raw数据缓存池

 

​       AVS_ GetPicBufferSize         AVS输入的YUV数据缓存池

 

​       VDEC GetPicBufferSize         VDEC输出的YUV帧存缓存池

 

​       VDEC GetTmvBufferSize         VDEC输出的Tmv数据缓存池

 

​       VENC GetRefBufferSize         VENC输出的重构帧YUV缓存池

 

​       VENC_ GetRefBufferSize        VENC参考帧大小

 

​       VENC_ GetRefPicInfoBufferSize VENC参考帧信息（pme、pmeinfo、 tmv）大小

 

​       HI MPI VB_ PhysAddr2Handle    用户态通过缓存块的物理地址获取其句柄。

 

​       Hi_ MPI VB_ Handle2PhysAddr     获取一个缓存块的物理地址。

 

​       HI_ MPI VB_ Handle2PoolId     获取一个缓存块所在缓存池的ID。

 

​       HIMPIVB_MmapPool              为一个视频缓存池映射用户态虚拟地址。

 

​       HI MPI VB_ MunmapPool         为一个视频缓存池解除用户态映射。

 

​       HI_ MPI _VB_ _GetBlockVirAddr获取一个视频缓存池中的缓存块的用户态虛拟地址。

#### 3.3sample中例程应用

想要实现从VI读入YUV格式的数据，保存到DDR中，然后VPSS或者SVP模块从DDR中调用数据进行处理。阅读sample中的traffic_capture例程。

内存分配的角度来看，首先启动参数设置的时候设置好OS内存和MMZ内存，在进行处理的时候在MMZ内存中申请初始化视频缓存池，每个缓存池又分为多个缓存块。创建好缓存池的情况下可以申请缓存块用来保存相应的图像信息。

 