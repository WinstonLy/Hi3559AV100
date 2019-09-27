### 一 概述

**I2C**总线是由Philips公司开发的一种简单、双向二线制同步串行总线。 它只需要两根线即可在连接于总线上的器件之间传送信息。 主器件用于启动总线传送数据，并产生时钟以开放传送的器件，此时任何被寻址的器件均被认为是从器件．

I2C 控制器为 Master 接口,完成 CPU 对 I 2 C 总线上连接的从设备的读写访问。芯片共20 个 I2C 控制器,主 SOC 子系统
12 个,Sensor Hub 子系统8个。

### 二 功能描述

- 芯片的 I 2 C 是 Master 接口,支持标准时序和非标准时序。
- 支持标准地址(7bit)和扩展地址(10bit)。
- 传输速率支持标准模式(100kbit/s)和快速模式(400kbit/s)。
- 支持 64 x 8bit 的 TX FIFO 和 64 x 8bit 的 RX FIFO。

### 三 I2C操作指南（内核下操作）

**准备**

- Linux 内核使用的SDK发布的kernel
- 文件系统，可以使用SDK发布的本地文件系统，也可以通过本地文件系统在挂载到NFS。

**操作过程**

- 启动单板，加载本地文件系统，或者通过本地文件系统进一步挂载到NFS。
- 加载内核，默认I2C相关模块已经去那部编入内核。
- 在控制台下运行I2C读写命令或者自行在用户台或者内核态下编写I2C读写程序，就可以通过对挂载到I2C控制器上的外围设备进行读写操作。

**用户态读写命令**

- 读命令`i2c_read`

  

  ```shell
  `i2c_read <i2c_num> <device_addr> <reg_addr> <end_reg_addr> <reg_width> <data_width> <reg_step>``
  i2c_num:I2C 控制器序号,经过i2c调试工具确定其控制器序号为0
  device_addr：外围设备地址，设为0x34
  reg_addr:读外围设备寄存器操作的开始地址
  end_reg_addr: 读外围设备寄存器操作的结束地址
  reg_width：外围设备的寄存器位宽，Hi3559AV100支持8/16/24/32bit，设为0x02（16bit）
  data_width:外围设备的数据位宽，Hi3559AV100支持8/16/24/32bit，设为0x01（8bit）
  reg_step：连续读寄存器，默认设置为1
  ```

- 写命令`i2c_write`

  ```shell
  `i2c_write <i2c_num> <device_addr> <reg_addr> <value> <reg_width> <data_width>`
  i2c_num:I2C控制器编号
  device_addr：外围设备地址
  reg_addr：写外围设备寄存器操作的地址
  value：写外围设备寄存器的数据
  reg_width：外围设备的寄存器位宽
  data_width：外围设备的数据位宽
  ```

- 用户台I2C读写程序

  此操作在用户态下通过I2C读写程序实现对I2C外围设备的读写操作

  1. 打开I2C总线对应的设备文件，获取文件描述符：

  ​     `fd  = open("/dev/i2c-0",O_RDWR)`;

  2. 进行数据读写

     `ioctl(fd, I2C_RDWR, &rdwr);
      write(fd, buf, (reg_width + data_width));`

### 四 I2C调试工具使用

在最基础的SDK版本（`Hi3559AV100_SDK_V2.0.1.0`）中I2C调试工具的组成不完整，代码有缺陷不能使用，新版的SDK（`Hi3559AV100_SDK_V2.0.2.0`）集成了I2C的调试工具，可以进行相关操作。对比两部分有关I2C的代码发现其中有部分函数功能未实现。如：

```shell
static u32 hibvt_i2c_func(struct i2c_adapter *adap)
{
	return I2C_FUNC_I2C | I2C_FUNC_10BIT_ADDR
		| I2C_FUNC_PROTOCOL_MANGLING
		| I2C_FUNC_SMBUS_WORD_DATA
		| I2C_FUNC_SMBUS_BYTE_DATA
		| I2C_FUNC_SMBUS_BYTE
		| I2C_FUNC_SMBUS_I2C_BLOCK;
}
```

在老版SDK中未实现这个函数，所以在使用i2cdetect的时候会出现不支持相关命令的提示，在新版的SDK中则不会出现这个问题。调用相关工具结果如下，根据调试的结果来调用`i2c_read`和`i2c_write`进行寄存器的读写设置。

```shell
//列举I2C BUS，即I2C的设备号
~ # i2cdetect -l
i2c-3   i2c             hibvt-i2c                               I2C adapter
i2c-1   i2c             hibvt-i2c                               I2C adapter
i2c-11  i2c             hibvt-i2c                               I2C adapter
i2c-8   i2c             hibvt-i2c                               I2C adapter
i2c-6   i2c             hibvt-i2c                               I2C adapter
i2c-4   i2c             hibvt-i2c                               I2C adapter
i2c-2   i2c             hibvt-i2c                               I2C adapter
i2c-0   i2c             hibvt-i2c                               I2C adapter
i2c-9   i2c             hibvt-i2c                               I2C adapter
i2c-10  i2c             hibvt-i2c                               I2C adapter
i2c-7   i2c             hibvt-i2c                               I2C adapter
i2c-5   i2c             hibvt-i2c                               I2C adapter

//查看I2C BUS i2c-0上面连接的所有设备
~ # i2cdetect -r -y 0
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- 1a -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- -- 
上面的1a就是连接的imx327的设备地址，采用7位地址，代码中设置为0x34,右移1位后就是0x1a。
```

**注意**

调用上述的调试工具的时候，需要运行例程（如`sample_vio`），这样才会给sensor上电，调用i2cdetect工具才会有正确结果，同时才能正确使用`i2c_read`和`i2c_write`,调用`i2c_read`结果如下

```shell
~ # i2c_read 0x00 0x34 0x3046 0x3046 0x02 0x01
*** Board tools : ver0.0.1_20121120 *** 
[debug]: {source/utils/cmdshell.c:168}cmdstr:i2c_read
i2c_num:0x0, dev_addr:0x34; reg_addr:0x3046; reg_addr_end:0x3046;                       reg_width: 2; data_width: 1; reg_step: 1. 

0x3046: 0x1
```

**telnet安装与使用**

因为在运行例程的时候不能退出界面，退出的话会导致sensor断电，不能进行调试，所以需要配置telnet协议。**telnet**协议是TCP/IP协议族中的一员，是Internet远程登录服务的标准协议和主要方式。 它为用户提供了在本地计算机上完成远程主机工作的能力。 在终端使用者的电脑上使用**telnet**程序，用它连接到服务器。

- 安装

  ```shell
  sudo apt-get update
  sudo apt-get install xinetd telnetd
  //重启机器或网络服务
  sudo /etc/init.d/xinetd restart
  ```

- telnet使用

  首先在开发板上输入 `telnetd&`开启telnet服务，然后在服务器上输入`telnet IP`即可远程登录到开发板上

- 登录过程中的密码

  用户名输入**root**，密码为默认直接回车

