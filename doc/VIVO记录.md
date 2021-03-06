## 一 MPP总体介绍

海思媒体处理平台（MPP）分为视频输入视频输入(VI)、视频处理(VPSS)、视频编码(VENC)、视频解码(VDEC)、视频输出(VO)、视频拼接(AVS)、音频输入(AI)、音频输出(AO)、音频编码(AENC)、音频解码(ADEC)、区域管理(REGION)等模块。

- VI 模块捕获视频图像,可对其做剪切、去噪等处理,并输出多路不同分辨率的图像数据。
- VPSS 模块接收 VI 和解码模块发送过来的图像,可对图像进行图像增强、锐化等处理,并实现同源输出多路不同分辨率的图像数据用于编码、预览或抓拍。
- VO 模块接收 VPSS 处理后的输出图像,可进行播放控制等处理,最后按用户配置的输出协议输出给外围视频设备。

## 二 系统控制

### 2.1 视频缓存池

视频缓存池主要向媒体业务提供大块物理内存管理功能,负责内存的分配和回收,充分发挥内存缓存池的作用,让物理内存资源在各个媒体处理模块中合理使用。在VIVO过程中，所有的视频输入通道都可以从公共视频缓存池中获取视频缓存块用于保存采集的图像，系统初始化之前必须配置公共视频缓存池。

### 2.2 系统绑定

数据接受者绑定数据源来建立两者的关联关系（只允许数据接收者绑定数据源），数据源生成的数据将自动发送给数据接收者。

### 2.3 VI和VPSS的工作模式

总共分为在线，离线，并行3个模式
            

​                  VI_CAP与VI_PROC                                    VI_PROC与VPSS
​             

​                                 data                                                               data

在线模式     VI_CAP-------->VI_PROC                           VI_PROC------->VPSS

离线模式    VI_CAP--->DDR--->VI_PROC                     VI_PROC--->DDR--->VPSS
                           

并行模式     VI_CAP与两个VI_PROC处于                     VI_CAP与两个VI_PROC处于并行模式，同时
     

​                        并行模式                                                两个VPSS也分别与VIPROC处于并行模式 

Hi3559AV100 VI PIPE工作模式 

PIPE ID               0       1       2      3      4        5       6        7    8-n

模式分布1         离线  离线 离线 离线  离线  离线  离线  离线  离线

模式分布2         在线  离线 离线 离线  离线  离线  离线  离线  离线

模式分布3         离线  离线 离线 离线  在线  离线  离线  离线  离线

模式分布4         在线      -       -       -     在线    -        -        -    不存在

模式分布5         并行      -       -       -         -      -        -        -    不存在

## 三 视频输入（VI）

### 3.1 概述

通过 MIPI Rx(含 MIPI 接口、LVDS 接口和 HISPI 接口),SLVS-EC,BT.1120,BT.656,BT.601,DC 等接口接收视频数据。VI 将接收到的数据存入到指定的内存区域,在此过程中,VI 可以对接收到的原始视频图像数据进行处理,实现视频数据的采集。



### 3.2 功能描述

VI在软件层次上划分为4个部分      ![](D:\workspace\Hi3559AV100\image\VI软件层次.png)             

- 输入设备（DEV），输入PIPE，物理通道（PHY_CHN），扩展通道（EXT_CHN）;      
- Hi3559AV100共有8个DEV，8个PIPE，1个PHY_CHN，8个EXT_CHN;
- 视频输入PIPE（有疑问：PIPE是什么？）：VI 的 PIPE 包含了 ISP 的相关处理功能,主要是对图像数据进行流水线处理,输出YUV 图像格式给通道。
- 视频扩展通道是物理通道的扩展,扩展通道具备缩放、裁剪、鱼眼矫正功能,它通过绑定物理通道,将物理通道输出作为自己的输入,然后输出用户设置的目标图像。


绑定关系：Hi3559AV100的DEV与MIPI的绑定关系是固定的
          

Hi3559AV100 DEV 与 MIPI/SLVS/BT.1120/BT.656/BT601/DC 接口的绑定关系

​             ![](D:\workspace\Hi3559AV100\image\DEV与接口绑定工具.png)

Hi3559AV100的MIPI接口与VI DEV的绑定关系是一一对应的，但用BT.1120的时候，第0组管脚时PIPE和VI DEV5绑定，才能接受数据。每个PIPE都可以与任意Dev绑定，但不能动态修改绑定关系。

VI分成两个物理子模块：VI_CAP和VI_PROC。VI_CAP完成数据捕获，将数据存到DDR（离线）或者送给VI_PROC（在线）。VI_PROC也分为离线模式（从DDR中读取数据）和在线模式（从VI_CAP接收数据）。VI模块最多能接受8路视频输入信号。功能框图如下所示：

![](D:\workspace\Hi3559AV100\image\VI功能框图.png)

VI_CAP特点：

- 输入最大分辨率为8192x8192,最大工作时钟600MHz。
  
- 支持最大数据位宽为14bit。
  
- 支持MIPI接口输入YUV422，YUV420格式。 


VI_PROC特点：

- 支持在线和离线两种工作模式。
  
- 内嵌ISP处理功能
  
-  输出数据格式支持:
                   

​    -- Semi-planar YCbCr 4:2:2 模式
​              

​    -- Semi-planar YCbCr 4:2:0 模式
​               

​    -- Semi-planar YCbCr 4:0:0 模式
​               

​    -- RAW 模式

### 3.3 视频接口之MIPI Rx

MIPI Rx:移动行业处理器接口MIPI Rx(Mobile Industry Processor Interface Receiver),通过低电压差分信号接收原始视频数据(BAYER RGB数据),并将其转化为DC(Digital Camera)时序后传递给下一级模块 VI_CAP。MIPI Rx支持MIPI D-PHY、LVDS、HiSPi等串行视频信号输入，串行视频接口可以提供更高的传输带宽，增强传输的稳定性。

MIPI  Rx功能框图及在系统中的位置

​    ![](D:\workspace\Hi3559AV100\image\MIPI RX.png)

注：Lane指差分数据对，MIPI Rx共16条Lane;

​       Link指Lane的分组,每个分组中有4对数据,MIPIRx有4个Link。
​    

MIPI Rx有以下特点：

- 支持 MIPI DPHY-ver1.2
  
- 可同时支持 8 路 sensor 输入
  
- 单路最多支持8-Lane MIPI D-PHY 接口,最大支持 2.5Gbps/Lane
  
- 单路最多支持16-Lane LVDS/ sub-LVDS /HiSPi 接口,最大支持 1.5Gbps/Lane
  
- 支持RAW8/RAW10/RAW12/RAW14/RAW16数据类型的解析
  
- 支持YUV420 8-bit/YUV420 10-bit/YUV422 8-bit/YUV422 10-bit/Legacy YUV4208-bit数据类型的解析
  
- 最多支持 4 帧 WDR,支持多种 WDR 时序
  
- 支持 LVDS/HiSPi 模式像素/同步码大小端配置
  
- 支持 Lane 数和 Lane 顺序可配置
  
-  通道 0 支持一拍双像素输出 
          

MIPI Rx包括4个D-PHY,每个PHY各自有两对差分随路时钟(CLK0/CLK1),每对时钟对应2对数据。因此MIPI Rx可以同时支持1~8路sensor输入。MIPI Rx只完成接口时序转换，不处理图像的数据格式。

MIPI Rx 的带宽有两部分限制:combo-PHY 的接口数据率和内部处理速度。
       

MIPI Rx支持D-PHY和CSI-2(Camera Serial Interface)。D-PHY 规定了物理层传输规范,CSI-2规定了Camera 输出数据包的格式和协议。
      

LVDS：通过同步码区分消隐区和有效区的数据，LVDS传输模式中,行场同步信号集成在数据流中,在数据流中的特殊码型SOF和EOF分别表示帧的起始和结束,SOL和EOL分别表示行的起始和结束。在数据流中,SOF/EOF/SOL/EOL由 4个字段构成,每个字段的位宽与像素数据保持一致,前3个字段为固定的基准码字,根据第4个字段来区分帧/行的起始或结束。
       

### 3.4 图像存储模式

YUV数据存储 ：

- semi-planar YCbCr 存储。
- 系统设定了视图区域后,对读入数据按照 semi-planar 方式存储,即亮度分量和色度分量分别存储在DDR中亮度存储空间和色度存储空间。
- 在 1 行内,亮度、色度分量各自连续存储。
- 连续 2 行之间的存储,可以通过系统定义的行首与行首之间的存储间隔参数 stride定义。
- 亮度和色度分量在 DDR 中的存储位置由起始地址 base_addr 来指示。      

RAW数据存储

- 按单分量方式存储

- 1行内,RAW数据连续存储。
- 连续2行之间的存储,可以通过系统定义的行首与行首之间的存储间隔参数stride定义。

- DDR 中的存储位置由起始地址 base_addr 来指示。      

## 四 视频输出（VO）

### 4.1 概述

VO主动从内存相应位置读取视频和图形数据,并通过相应的显示设备输出视频和图形。
                     

Hi3559AV100支持的显示/回写设备，视频层和图形层r如下图：

​     ![](D:\workspace\Hi3559AV100\image\芯片支持显示回写视频层.png)

注：       

- DHD0：Device HD0，超高清设备0.支持4K（38400x2160）的时序。
  
- DHD1：Device HD1，高清设备1。
  
- VHD0:Video layer of HD0,超高清视频层 0,隶属于 DHD0。
  
- VHD1:Video layer of HD1,高清视频层 1,隶属于 DHD1。
  
- VHD2:Video layer of HD 2,高清视频层 2,隶属于 DHD0,用作 PIP 层。
  
- WD:Write Back Channel Device,回写通道设备。
  
- 图形层 G3:Graphic layer 3,用作鼠标层,DHD0 和 DHD1 中均有此项,但只能绑定其中一个设备,
     G3 默认绑定在 DHD1 上。
     
- 输出时序：HDMI 8K@30;MIPI Tx 1080P60;BT.1120 1080P60;BT.656 1080P30；LCD 1080P60。
  
- 回写功能:捕获视频层和设备级的视频数据,可用于显示和编码。
                                  

### 4.2 输出接口

超清HDMI2.0 

- 支持8bit/10bit输出；
- 时钟频率为74.25MHz~594MHz；
- 最大支持2160P@60fps YCbCr444/RGB输出；
- 支持数据来自DHD0。

高清MIPI Tx 

- 支持10bit输出;
- 最大支持1080P@60fps数据传输和显示；
- 还支持720P@60fps。

高清BT.1120 

- 支持YCbCr422 8bit输出;
- 支持720P，1080P;
- 16bit数据，默认Y在高位，C在低位，YC位置可以互换；
- 支持bypass模式，支持输出随路时钟反相。

标清BT.656  

- 支持YCbCr422 8bit输出;
- 支持以下典型输出时序:NTSC，PAL。

LCD输出接RGB 

- 支持8bit/6bit串行RGB输出;
- 支持16bit/24bit并行RGB输出;
- 支持RGB串行输出；
- 支持RGB并行输出。