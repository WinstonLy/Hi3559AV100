### 一,重要概念

#### 	1.基本概念

​	LVDS：低压差分信号，通过同步码区分消隐区和有效数据

​	Lane：用于连接发送端和接收端的一对高速差分线，可以是时钟Lane，也可以	是数据Lane

​	Link：发送端和接收端之间的时钟Lane和至少一个数据Lane组成一个Link

​	同步码：LVDS使用同步码区分消隐区和有效数据，有两种同步方式

​	MIPI Rx是一个支持多种差分视频输入接口的采集单元，通过combo-PHY接收数	据，Hi3559AV100最大支持8 Lane    	MIPI 输入或16 Lane LVDS 输入，最大	对接8个sensor。MIPI Rx包括4个D-PHY，每个PHY各自有两队差分随路时钟（CLK0/CLK1），没对时钟对应2对数据。

​	MIPI Rx提供对接sensor时序的功能。提供ioctl接口。

#### 2.MIPI Rx概述

​	MIPI Rx分为combo-PHY和Controller两部分，功能框图及在系统中的位置如下	所示：

![1565073521100](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1565073521100.png)

​	MIPI Rx的como-PHY将差分串行数据转换为并行数据，MIPI Rx控制器将并行数	据拆分，拼接，然后提取同步码，解	析出像素数据。

#### 3.ioctl

​	应用层的ioctl函数传入的cmd和arg参数会直接传入驱动层的ioctl接口，ioctl接口	的命令有一定规范详细查看ioctl-			

​	number.txt文件。

> ​	`int (*ioctl) (struct inode *inode,struct file *filp,unsigned int cmd,unsigned long arg);`
>
> ​	`inode与filp两个指针对应于应用程序传递的文件描述符fd，这和传递open方法的参数一样。`
>
> ​	`	cmd 由用户空间直接不经修改的传递给驱动程序`
> `arg 可选。`

​	cmd命令码由  魔数 序号 方向 数据尺寸  构成，常用的几个命令如下：

> ​	`//nr为序号，datatype为数据类型,如int`
>
> ​	`_IO(type, nr ) //没有参数的命令`
>
> ​	`_IOR(type, nr, datatype) //从驱动中读数据
> `
>
> ​	`_IOW(type, nr, datatype) //写数据到驱动`
>
> ​	`_IOWR(type,nr, datatype) //双向传送宏`

### 二,接口设置流程

#### 1.配置流程

​	1）设置Lane分布模式   HI_MIPI_SET_HS_MODE

​	2）打开多路MIPI时钟   HI_MIPI_ENABLE_MIPI_CLOCK

​	3）复位多路SENSOR 所对接的MIPI Rx    HI_MIPI_RESET_MIPI 

​	4）打开多路SENSOR所连接的时钟   HI_MIPI_ENABLE_SENSOR_CLOCK

​	5）复位对接的所有SENSOR    HI_MIPI_RESET_SENSOR

​	6）配置MIPI Rx 设备属性   HI_MIPI_SET_DEV_ATTR

​	7）撤销复位多路SENSOR所对接的MIPI Rx    HI_MIPI_UNRESET_MIPI

​	8）撤销复位对接的所有SENSOR    HI_MIPI_UNRESET_SENSOR

​	注：

（a）在SVP第一种RFCN例程中，修改第6步中的属性，

​	SAMPLE_COMM_VI_SetMipiAttr

​		SAMPLE_COMM_VI_GetComboAttrBySns

​		    case SONY_IMX334_SLAVE_MIPI_8M_30FPS_12BIT:

 		   case SONY_IMX334_MIPI_8M_30FPS_12BIT:

​				MIPI_4lane_CHN0_SENSOR_IMX334_12BIT_8M_NOWDR_ATTR

​					修改其中的input_mode为INPUT_MODE_LVDS

​					修改其中的lvds_attr结构体中的参数（LVDS相关参数）

​	（b）Lane id 的配置

​		在之前的过程中我们已经配置了Lane的分布模式。

                       Mode   DEV0   DEV1 DEV2  DEV3  DEV4    DEV5   DEV6    DEV7

![1565080194649](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1565080194649.png)

​	lane_id数组的索引号表示sensor的LANE ID，lane_id数组的值表示MIPI的LANE ID。对接sensor时，未使用的lane将其	对应的lane_id配置为-1。假设sensor Lane管脚为Lane 0 1 2 3，MIPI Lane管	脚为 Lane 4 5 6 7。MIPI的最大Lane数	为8，我们认为SENSOR的Lane数目也有8个，由于 sensor实际只有4个Lane，只输出数据到MIPI的4个Lane，需要将	SENSOR未连接的或者不存在的Lane的lane_id配置为-1，所以所以 lane_id配置如下：

​     lane_id = {4,5,6,7,-1,-1,-1,-1}

​	(c)修改后的结构体

​	`combo_dev_attr_t MIPI_4lane_CHN0_SENSOR_IMX334_12BIT_8M_NOWDR_ATTR`

​	`{`

​		`.devno = 0,`

​		`.input_mode = INOUT_MODE_LVDS,`

​		`.data_rate = MIPI_DATA_RATE_X1,`

​		`img_rect = {0, 0, 3840, 2160},`

​		`{`

​				`.lvds_attr =` 

​				`{`

​					`DATA_TYPE_RAW_12BIT,`

​					`HI_WDR_MODE_NONE,`

​					`LVDS_SYNC_MODE_SOF,`

​					`.sync_type = LVDS_VSYNC_NORMAL,`

​					`.fid_type = LVDS_FID_NONE,`

​					`LVDS_ENDIAN_LITTLE,`

​					`LVDS_ENDIAN_LITTLE,`

​					`{4, 5, 6, 7, -1, -1, -1, -1}`

​					`同步码待设置`

​				`}`

​		`}`	

​	`}`

#### 2.退出流程

​	1）复位多路对接的SENSOR     HI_MIPI_RESET_SENSOR

​	2）关闭多路SENSOR所连接的时钟    HI_MIPI_DISABLE_SENSOR_CLOCK

​	3）复位多路SENSOR所对接的MIPI Rx    HI_MIPI_RESET_MIPI

​	4）清除多路SENSOR所对接的MIPI Rx设备的配置     HI_MIPI_CLEAR

​	5）关闭多路MIPI时钟     HI_MIPI_DISABLE_MIPI_CLOCK

### 三,LVDS接口数据格式

#### 1.概念介绍

​	LVDS 通过同步码区分消隐区和有效数据。

​	在LVDS传输模式中，行场同步信号集中在数据流中，在数据流中的特殊码型 SOF和EOF分别表示帧的起始和结束， 	SOL和EOL分别表示行的起始和结束。在数据流中，SOF/EOF/SOL/EOL由4个字段构成，前三个为固定的基准码字，	根据第四个字段来区分帧/行的其实或结束。

#### 2.传输方式

​	以4个Lane为例，LVDS同步码和像素数据在各个Lane上传输方式如下，H表示	同步码，P表示像素。

​	LVDS同步码和图像传输方式1

![1564975530457](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1564975530457.png)

​	LVDS同步码和图像传输方式2

![1564975628764](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1564975628764.png)

#### 3.同步方式

​	使用SOF/EOF表示帧起始和结束，使用SOL/EOL表示行的起始和结束。SOF标识有效区的第一行起始，EOF标识有效	区最后一行的结束，其他有效区分别用SOL和EOL作为起始和结束。

![1564968618805](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1564968618805.png)

​	使用SAV（invalid）EAV（invalid）表示消隐区的无效数据开始和结束，使用SAV（valid）和EAV（valid）表示有效像	素数据的开始和结束。每个同步码由4个字段组成，每个字段的位宽和像素数据位宽保持一致。

![1564968499281](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1564968499281.png)

#### 4.具体像素保存形式（FAQ）

![1564991326315](C:\Users\Vigny\AppData\Roaming\Typora\typora-user-images\1564991326315.png)

### 四,LVDS模式配置

​	LVDS模式下需要配置RAW DATA类型、数据大小端、同步方式、WDR类型和图	像宽高等寄存器。LVDS模式依靠同	步码识别帧/行同步信息，根据 RAW DATA类	型的不同，同步码可以为8/10/12/14/16-bit。

​	LVDS模式的软件操作流程如下：（FAQ寄存器如何配置）

1. 上电启动；
2. 根据使用场景将MISC_CTRL130寄存器中相应通道的mipi_work_mode配置为 LVDS模式；
3. 配置CRG寄存器中的PERI_CRG61，打开mipi_bus_clken以及对应通道的mipi_pix_clken。配置mipi总线软复位，撤销复位；配置相应通道pix_core复位，撤销复位；
4. 配置CRG寄存器中的PERI_CRG69，配置sensor复位，撤销复位；打开sensor时钟门控，并配置时钟频率；
5. 配置CRG寄存器中的PERI_CRG60，选择MIPI RX通道时钟频率。
6. 配置接收数据类型，WDR模式，图像宽高（LVDS模式下，配置的宽度是图像实际宽度除以Lane数-1），同步头，Lane ID等信息；
7. 配置PHY的工作模式（PHY_MODE_LINK* ），PHY通道延迟调节（PHY_SKEW_LINK* ），PHY通道使能（PHY_EN_LINK* ），PHY均衡调节（PHY_EQ_LINK* ），PHY性能调节（PHY_CFG_LINK* ）；配置LVDS模式Lane同步头信息（PHY_SYNC_CODE*_LINK *)；
8. 配置系统控制寄存器。场景模式选择（HS_MODE_SELECT），PHY_EN，LANE_EN，打开PHY_CIL_CTRL，选择PHYCFG_MODE（对于LVDS模式，应选择1）；
9. 配置对应的PHYCFG_EN；
10. 配置sensor序列。

