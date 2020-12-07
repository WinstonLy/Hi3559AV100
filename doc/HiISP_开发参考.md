### 一 ISP概述

#### 1.1 概述

​	ISP通过一系列数字图像处理算法完成对数字图像的效果处理。包括逻辑部分和运行在其上的firmware。

#### 1.2 功能描述

####  1.2.1 架构

​	ISP 的 Firmware 包含三部分，一部分是 ISP 控制单元和基础算法库，一部分是AE/AWB 算法库，一部分是 sensor 库。Firmware 设计的基本思想是单独提供 3A 算法库，由 ISP 控制单元调度基础算法库和 3A 算法库，同时 sensor 库分别向 ISP 基础算法库和 3A 算法库注册函数回调，以实现差异化的 sensor 适配。

#### 1.2.2 开发模式

​	用户需要根据 ISP 基础算法库和 3A 算法库给出的sensor 适配接口去适配不同的 sensor。每款sensor对应两个主要文件：

- sensor_cmos.c

  主要实现ISP需要的回调函数，这些回调函数包括了sensor的适配算法。

- sensor_ctrl.c

  sensor的底层控制驱动，主要实现sensor的读写和初始化动作。

#### 1.2.3 软件流程

ISP作为前段采集部分，需要和视频采集单元（VIU）协同工作。ISP 初始化和基本配置完成后，需要 VIU 进行接口时序匹配。一是为了匹配不同 sensor 的输入时序，二是为 ISP 配置正确的输入时序。

ISP_firmware使用流程如下图所示：

![ISP_firmware使用流程](/media/winston/C14D581BDA18EBFA/workspace/Hi3559AV100/image/ISP_firmware使用流程.png)



### 二 系统控制

#### 2.1 功能概述

