这周主要的工作是进行LVDS接受信号的验证，由于之前的sensor不支持lvds输出，所以我们购买了IMX327这一款sensor，由于Hi3559AV100自带的SDK中没有IMX327的相应驱动，我从Hi3516的SDK中找到的imx327的相应驱动代码，然后想将其移植到Hi3559a中去。在进行一系列的修改之后，现在的情况是驱动部分编译没有问题，但是在运行相应例程的时候出现了一些问题，正在解决。

### 一 sensor调试指南

- 硬件准备：验证是否可以读写Sensor寄存器，利用i2c_read/i2c_write命令，或者ssp_write/ssp_read命令，测试寄存器读写。
- 准备sensor驱动：利用一款规格相近的驱动进行修改，编译出Sensor库。具体可参照xxx_cmos.c和xxx_sensor_ctl.c进行修改。修改相应函数使得分辨率，帧率可以被正确设置，在加载ko的脚本中修改sensor时钟配置，I2C/SPI接口pin mux，vi时钟，isp时钟等寄存器修改。
- sensor初始化序列：实现void sensor_init函数，参考imx327的数据手册实现这个函数。在xxx_sensor_ctl填写寄存器的基地址sensor_i2c_addr，地址的比特位宽sensor_addr_byte,寄存器的比特位宽信息sensor_data_byte。
- sensor输出：本部分是基于 mpp 目录下的 sample 做整个通路的输出说明。主要在已完成了 sensor 序列的前提下做的。其步骤主要包括:MIPI、VI、ISP 以及 VPSS 的配置。这些配置可以参考已 sensor 的配置进行简单修改即可。
- 验证：运行相应的例程进行验证，如最简单的sample_vio例程。

### 二 遇到的问题

#### 2.1 LVDS接口设置

根据imx327的数据手册，输出通过I2C协议，输出接口的切换通过OMODE pin 设置，输出模式的接口设置寄存器通过OPORTSEL设置。具体设置如下：

接口切换

![Interface Switching](/home/winston/workspace/Hi3559AV100/image/Interface Switching.png)

​		输出模式设置

![Output Register](/home/winston/workspace/Hi3559AV100/image/OutputRegister.png)                    

#### 2.2 Chip ID怎么选择？

阅读数据手册中的Setting Register Using Serial Communication ，通过I2C写寄存器，具体怎么实现？

通过设置相应的寄存器值选择chip id，例如：当为写寄存器的时候，寄存器Chip ID = 02h，读寄存器的时候，寄存器Chip ID =82h，其他的也有相应的对应值。根据寄存器映射指定地址。使用通信方法时指定连续地址，地址自动递增先前发送的地址。

#### 2.3 在将改过的驱动加入到Hi3559AV的SDK中的时候出现`loading out-of-tree module taints kernel`的问题。

在网上查阅相关问题，定位的问题可能是出现kernel不匹配的造成文件开发板上的系统读取文件出现了权限问题。我将重新编译的boot，kernel，文件系统重新烧写进开发板，成功的解决了上述问题。

另外在烧写的时候利用最新版本的Hitool工具烧写的时候可能遇到TFTP超时的问题，解决办法是在确认ip配置正确的前提下，修改烧写工具的TFTP属性设置，同时关闭防火墙。

#### 2.4 管脚的设置

- OMODE pin ，选择CSI-2或者LVDS
- XMASTER pin，设置为High处于Slave Mode，设置为Low处于Master Mode

> 

#### 2.6 运行例程的时候出现了如下所示的问题

```shell
/mnt/mpp/sample/vio # ./sample_vio 0 0
[SAMPLE_COMM_VI_SetMipiAttr]-1964: ============= MipiDev 0, SetMipiAttr enWDRMode: 0
linear mode
[Func]:DrcCheckCmosParam hibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
[Line]:2450 [Info]:Invalid u16AutoStrength!
[Func]:DemosaicCheckhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
CmosParam [Line]:408 [Info]:Invalid au8NonDirMFDetailEhcStr[0]:48hibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE errorhibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE error!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE error!
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE error!
hibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE error!
hibvt-i2c 12110000.i2c: wait idle abort!, RIS: 0x611
[Func]:imx327_write_register [Line]:162 [Info]:I2C_WRITE error!
===IMX327 1080P 30fps 12bit LINE Init OK!===
[SAMPLE_COMM_ISP_Thread]-322: ISP Dev 0 running !
[SAMPLE_COMM_VO_StartChn]-544: u32Width:3840, u32Height:2160, u32Square:1
---------------press Enter key to exit!---------------


```

1. 