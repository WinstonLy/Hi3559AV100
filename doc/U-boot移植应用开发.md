### 														U-boot移植应用开发

当选用的外围芯片的型号与单板上外围芯片的型号不同时，需要修改 U-boot 配置文件，主要包括存储器配置、管脚复用。

​	`（Hi3559A╱C V100 U-boot 移植应用开发指南.pdf）`

**一 U-boot主要目录结构**

| 目录名                       | 描述                                              |
| ---------------------------- | ------------------------------------------------- |
| `arch`                       | 各种芯片架构的相关代码、U-boot 入口代码。         |
| `board`                      | 各种单板的相关代码，主要包括存储器驱动等。        |
| `board/hisilicon/hi3559av10` | `Hi3559AV100` 单板相关代码。                      |
| `arch/xxx/li`                | 各种体系结构的相关代码，如 ARM、MIPS 的通用代码。 |
| `include`                    | 头文件。                                          |
| `include/config`             | 各种单板的配置文件。                              |
| `common`                     | 各种功能（命令）实现文件。                        |
| `drivers`                    | 网口、Flash、串口等的驱动代码。                   |
| `net`                        | 网络协议实现文件。                                |
| `fs`                         | 文件系统实现文件。                                |
| `product/hiupdate`           | SD 卡升级、`USB` 升级功能实现                     |
| `product/hios`               | `dec`、`hdmi `接口、`vo`、`mipi `功能实现         |
| `product/hiaudi`             | 音频功能实现                                      |



### 二 移植U-boot

#### 2.1 编译U-boot

当所有移植步骤完成后开始编译U-boot

- 配置编译环境

  - 当启动介质是 `SPI-Nor Flash` 或 `SPI-NAND Flash `时，使用配置命令：

    `make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_defconfig `

  - 当启动介质是并口` NAND Flash `时，使用配置命令：
    `make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_nand_defconfig `

  - 当启动介质是 `eMMC `时，使用配置命令：
    `make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_emmc_defconfig `

- 编译U-boot

  `make CROSS_COMPILE=aarch64-himix100-linux- -j 20 `

编译成功后在`U-boot`目录下生成`u-boot.bin`,并不是在淡斑上执行的`U-boot`镜像

#### 2.2 配置`DDR`存储器

在 Windows 下打开 `SDK` 中的`osdrv/tools/pc/uboot_tools/`目录下的配置表格。当选用不同的 `DDR SDRAM` 时，需要针对不同器件的特性，对配置工作表中的 `DDR` 相关标签页进行修改。

#### 2.3 配置管教复用

我们并没有改变管脚复用，此过程跳过。

#### 2.4 生成最终使用的U-boot镜像

生成步骤如下所示：

-  完成配置表格的修改后，保存表格；

- 单击表格第一个标签页上的按钮`Generate reg bin file`或者使用 `hiregbin `工具（详细使用方法请参考 `osdrv/ tools/pc/uboot_tools/ hiregbin-v5.0.1.tgz` 压缩包里的 `readme `文件），生成临时文件` reg_info.bin`；

- 将临时文件` reg_info.bin` 拷贝到 `SDK `中的`osdrv/opensource/uboot/u-boot-2016.11/`目录下，并命名为`：.reg`，然后执行命令：

  `make CROSSCOMPILE=aarch64-himix100-linux- u-boot-z.bin `

生成的` u-boot-hi3559av100.bin` 就是能够在单板上运行的` u-boot `镜像。

#### 2.5 `hiregbin`工具使用

1. 在命令行下，进入`hiregbin`所在目录；

2. 输入命令如下：

   `./hiregbin [excelFile] [outputFile]`

   `excelFile`、`outputFile`为必须参数，由用户指定。
   示例：
   `./hiregbin ./Hi35**.xlsm ./reg.bin`

3. 等待制作完成。

### 三 烧写U-boot

如果单板已有u-boot，课通过串口或网口与服务器相连直接更新U-boot。如果没有则需要用HiTool工具进行烧写。工具烧写参考《`HiBurn `工具使用指南》。

#### 3.1 `Flash` 的 `U-boot`烧写方法

- `SPI-Nor Flash`烧写方法

  在内存中运行起来之后在超级终端输入：

  ```shell
  hisilicon# mw.b 0x42000000 ff 0x100000 				/* 对内存初始化*/ 
  hisilicon# tftp 0x42000000 u-boot-hi3559av100.bin 	/*U-boot下载到内存*/ 
  hisilicon# sf probe 0 								/*探测并初始化SPI-Nor flash*/ 
  hisilicon# sf erase 0x0 0x100000					/*擦除 1M大小*/ 
  hisilicon# sf write 0x42000000 0x0 0x100000 		/*从内存写入SPI-Nor Flash*/ 
  ```

  上述步骤操作完成后，重启系统可以看到 U-boot 烧写成功。

- `NAND Flash/SPI-Nand Flash`的烧写方法

  在内存中运行起来之后在超级终端中输入：

  ```shell
  hisilicon# nand erase 0 0x100000 					/*擦除 1M大小*/ 
  hisilicon# mw.b 0x42000000 0xff 0x100000 			/* 对内存初始化*/ 
  hisilicon# tftp 0x42000000 u-boot-hi3559av100.bin   /*U-boot下载到内存*/ 
  hisilicon# nand write 0x42000000 0 0x100000 		/*从内存写入NAND Flash*/ 
  ```

  重启系统可以看到 U-boot 烧写成功。

#### 3.2 `eMMC`的`U-boot`烧写方法

在内存中运行起来之后在超级终端中输入：

```shell
hisilicon# mw.b 0x42000000 0xff 0x80000				 /* 对内存初始化*/ 
hisilicon# tftp 0x42000000 u-boot-hi3559av100.bin	 /* U-boot下载到内存*/ 
hisilicon# mmc write 0 0x42000000 0 0x400			 /*从内存写入eMMC*/ 

mmc write 命令说明：
格式：mmc write <device num> addr blk# cnt
<device num>：设备号
addr：源地址
blk#：目的地址的块序号
cnt：块的数目，块大小为 512 字节
```

重启系统可以看到 U-boot 烧写成功。