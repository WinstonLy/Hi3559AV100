## 一 Atlas 200 DK描述

### 1. 产品简介

华为Atlas 200 DK开发者套件 Atlas 200 Developer Kit（缩写：Atlas 200 DK ）是以华为Ascend 310芯片为核心的一个	开发者板形态产品，并为开发者提供一站式开发套件，助力开发者快速进行AI应用程序的开发。主要功能是将Atlas 200	的接口对外开放，方便用户快速简捷的使用Atlas 200，可以运用于平安城市、无人机、机器人、视频服务器等众多领域	的预研开发。

Atlas 200 AI加速模块（简称Atlas 200）是一款高性能的AI智能计算模块，集成了海思Ascend 310 AI处理器，可以实现	图像、视频等多种数据分析与推理计算，可广泛用于智能监控、机器人、无人机、视频服务器等场景。

Ascend 310是一款华为专门为图像识别、视频处理、推理计算及机器学习等领域设计的高性能、低功耗AI芯片。芯片内	置2个AI core，可支持128位宽的LPDDR4x，可实现最大16TOPS（INT8）的计算能力。

Atlas 200 DK系统框图	如下所示，其中Atlas 200 DK开发者板主要包含Hi3559 Camera模块以及Atlas 200 AI加速模	块，开发工具MindSpore Studio所在PC通过USB接口或者网线与Atlas 200 DK开发者板连接。

![](/home/winston/图片/atlas_200_dk/Atlas_200_DK系统框图.png)

### 2. 产品特点

- 可提供16TOPS（INT8）的峰值计算能力。
- 支持两路Camera输入，两路ISP图像处理，支持HDR10高动态范围技术标准。
- 支持1000M以太网对外提供高速网络连接，匹配强劲计算能力。
- 通用的40-pin扩展接口（预留），方便产品原型设计。
- 支持5V~28V宽范围直流电源输入。

### 3. 产品规格

![](/home/winston/图片/atlas_200_dk/基本规格.png)

### 4. MindSpore Studio简介

MindSpore Studio是一套基于华为NPU（Neural-network Processing Unit）开发的AI全栈开发平台，包括基于芯片的算	子开发、以及自定义算子开发，同时还包括网络层的网络移植、优化和分析，另外在业务引擎层提供了一套可视化的AI	引擎拖拽式编程服务，极大的降低了AI引擎的开发门槛，全平台通过Web的方式向开发者提供以下4项服务功能。

- 针对算子开发

  MindSpore Studio提供全套的算子开发、支持真实环境运行，支持针对动态调度的异构程序的可视化调试，支持第三方算子开发，极大的降低了基于华为自研NPU的算子开发门槛，提高算子开发效率，有效提升产品竞争力。

- 针对网络层的开发

  MindSpore Studio集成了离线模型转换工具（OMG）、模型量化工具、模型运行Profiling分析工具和日志分析工具，极大的提升了网络模型移植和分析优化的效率。

- 针对AI引擎开发

  MindSpore Studio提供了AI引擎可视化拖拽式编程以及大量的算法代码自动生成技术，极大的降低了开发者的门槛，并且预置了丰富的算法引擎，如：Resnet18等，大大提高了用户AI算法引擎开发及移植效率。

- 针对应用开发

  MindSpore Studio内部集成了各种工具如Profiler、Compiler等，为用户提供图形化的集成开发环境，通过MindSpore Studio进行工程管理、编译、调试、仿真、性能分析等全流程开发，从而提高开发效率。

  开发流程如下图所示：

![](/home/winston/图片/atlas_200_dk/开发流程.png)

## 二 MindSpore Studio工具部署

### 1. 简介

- indSpore Studio是一套基于华为自研NPU开发的AI全栈开发平台，包括基于芯片的算子开发、调试、调优以及第三方算子开发，同时还包括网络层的网络移植、优化和分析，另外在业务引擎层提供了一套可视化的AI引擎拖拽式编程服务，极大的降低了AI引擎的开发门槛，全平台通过Web的方式向开发者提供一系列的服务。
- DDK（Device Development Kit）为用户提供基于NPU的数字开发者套件。DDK可以用于构建相关工程的编译环境。不同的发布包里集成了不同NPU形态的DDK。当前版本的DDK集成了TE、DVPP、流程编排等组件的依赖库和头文件，用户可以通过makefile编译相应的工程文件。

### 2. 环境准备

- 服务器端需要进行配置

- 客户端可以通过浏览器访问MindSpore Studio环境。需安装64位，67.0.3396.87及以上版本的**Chrome**浏览器。

#### 2.1 服务器端

##### （1）单用户场景推荐

​	内存≥4G，操作系统为Ubuntu16.04.3，安装好之后需要将内核版本升级到4.18以上。

##### （2）安装依赖

​	Python2：2.7.5，Python3：3.5.2，JDK：1.8.0_171+

##### （3）背景信息

- 当前工具只支持单用户安装并操作，不支持多用户同时安装或使用。

- 目前只支持普通用户安装，不支持root用户安装。

  若没有普通用户，则先用root用户新建一个，操作方法如下：

  > - 执行以下命令创建普通用户并设置普通用户的$HOME目录。
>
  >   `useradd -d /home/username -m username`
  >
  > - 执行以下命令设置密码。
  >
  >   `passwd username`
  >
  > - 执行以下命令设置权限pwd。
  >
  > - 进入“/home”目录。
  >
  >   `chmod 750 /home/username`

##### （4）配置源

​    请确保安装MindSpore Studio服务器能够连接网络。

- 切换到root用户为普通用户设置sudo apt-get权限。

  a. 执行以下命令打开“/etc/sudoers”文件：

```
	su root
	chmod u+w /etc/sudoers
	vi /etc/sudoers
```

​	b. 在该文件“# User privilege specification”下面增加如下内容：

```
	username ALL=(ALL:ALL) NOPASSWD:SETENV:/usr/bin/apt-get
```

​	username为执行安装脚本的普通用户名。

​	注意：请确保“/etc/sudoers”文件的最后一行为“#includedir /etc/sudoers.d”，如果没有该信息，请手动添加。

​	c. 添加完成后，执行:wq!保存文件。

​	d. 执行以下命令取消“/etc/sudoers”文件的写权限：

```
	chmod u-w /etc/sudoers
```

- 配置Ubuntu16.04的源，源的官方地址在“/etc/apt/sources.list”文件中。
- 执行sudo apt-get update命令检查源是否可用，如果根据“/etc/apt/sources.list”文件中的源无法下载相关依赖，建议用户配置可用代理或者搜索可用的源。

##### （5）安装依赖

​	请切换到普通用户执行如下操作。

​	1.执行以下命令安装相关依赖（如下命令请分别执行，单行命令如果出现换行，请将命令复制到word或者记事本中，在	换行处加入空格，再合成一行复制到服务器中执行）。

```
sudo apt-get install gcc g++ cmake curl libboost-all-dev unzip haveged libatlas-base-dev python-skimage python3-skimage python-pip python3-pip liblmdb-dev libhdf5-serial-dev libsnappy-dev libleveldb-dev make graphviz autoconf libxml2-dev libxml2 sqlite3 python libzip-dev libssl-dev
```

#### 2.2 安装JDK。

##### a. 执行sudo apt-get install -y openjdk-8-jdk命令安装JDK。

- 若用户本地已经安装JDK，使用**java -version**命令查看版本号，若版本号低于1.8.0_171，则请卸载JDK，然后通过[配置源](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0147565183.html)中的方法重新下载源，并安装JDK。
- 若用户通过[配置源](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0147565183.html)中的源以及[2.a](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0170954053.html#ZH-CN_TOPIC_0170954053__li7777363012)中的命令安装的JDK，通过**java -version**命令查看版本号，若版本号低于1.8.0_171，则使用**sudo apt-get update**更新源。
- 若通过以上两种方法更新源后，若版本号仍低于1.8.0_171，则进入[Oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jce-all-download-5170447.html)下载**jce_policy-8.zip**文件，将该包中的**local_policy.jar**和**US_export_policy.jar**文件，替换掉“%JAVA_HOME%\jre\lib\security”中的相应文件（%JAVA_HOME%为环境变量，该环境变量所指地址可以在**.bashrc**文件中查看）。

##### b. 设置环境变量。

​	完成JDK安装后，需要配置JAVA_HOME环境变量， 的安装及运行都依赖该环境变量，如若不设置该环境变量，		 	MindSpore Studio

- 在任何目录下执行**vi ~/.bashrc**命令，打开**.bashrc**文件。

- 在文件的最后一行后面添加如下内容。

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
export PATH=$JAVA_HOME/bin:$PATH
```

​	注：“JAVA_HOME”为JDK的安装目录，若用户已经配置了JDK，请根据安装目录的实际情况进行修改。若根据上述步	骤安装的JDK，则安装目录不用修改。

- 执行**:wq!**命令保存文件并退出。
- 执行**source ~/.bashrc**命令使环境变量生效。
- 执行echo $JAVA_HOME命令检查环境变量设置，回显信息如下：

```
/usr/lib/jvm/java-8-openjdk-amd64
```

- 执行which jconsole命令检查JDK安装。如果输出如下回显信息表示安装成功，如果未输出如下回显信息表示JDK安装失败。

```
/usr/lib/jvm/java-8-openjdk-amd64/bin/jconsole
```



### 3. 软件包准备

登录https://github.com/Ascend在tool目录下下载相关软件包

#### 3.1 设置安装目录权限

​	使用MindSpore Studio的安装用户，在Linux系统的$HOME目录下创建放置安装包的目录，例如：director。使用命令	为：

```
mkdir director
```

​	director目录对于普通用户必须具有读写和执行的权限，如果没有相关权限，请使用**su root**切换到root用户执行如下命	令：

```
chown username:usergroup director
chmod 750 director
```

- 只对*director*文件夹设置750权限即可。

#### 3.2 上传安装包

​	使用安装MindSpore Studio的普通用户将如下文件上传到*director*目录下：

- mini_mindSpore_studio_Ubuntu.rar：MindSpore Studio安装包。

- mini_mindSpore_studio_Ubuntu.rar.asc：MindSpore Studio安装包校验文件。

- MSpore_DDK****tar.gz：DDK安装包。

- MSpore_DDKtar.gz.asc：DDK安装包校验文件。

  注意：MindSpore Studio安装包与DDK安装包需要放在同一个目录。

#### 3.3 软件完整性检测

​	为了防止软件包在传输过程中由于网络原因或存储设备原因出现下载不完整或文件破坏的问题，在执行安装前，建议您	对软件包的完整性进行校验。

​	在安装包所在目录*director*下，执行如下操作：

##### a. 配置OpenPGP公钥信息

前提是Linux系统已经安装GnuPG 工具。若已经安装GnuPG 工具，在 Shell 中输入 **gpg --version**命令查看相应信息，若没有安装GnuPG 工具，则在GnuPG 的官方网站http://www.gnupg.org/，按照网站的指引，完成工具安装。

配置公钥步骤如下：

- 获取公钥文件，进入https://support.huawei.com/enterprise/zh/tool/pgp-verify-TL1000000054，单击下载链接下载KEYS.txt。

- 将下载的**KEYS.txt**MindSpore Studio例如到"/home/*username*/openpgp/keys"新建目录中。

- 导入公钥文件，执行如下命令进入公钥文件所在的目录导入公钥。

```
gpg --import "/home/username/openpgp/keys/KEYS.txt"
```

- 执行如下命令查看公钥导入结果。

```
gpg --fingerprint
```

- 验证公钥，OpenPGP 公钥的合法性需要根据公钥的 ID、指纹、uid 等信息与发布公钥的主体进行合法性验证。当前对外发布的OpenPGP公钥信息如下：

##### b. 测软件包完整性

使用MindSpore Studio安装用户分别执行如下命令，检测MindSpore Studio和DDK软件包是否合法完整。

- MindSpore Studio

  ```
  gpg --verify "mini_mindspore_studio_*.rar.asc"
  ```

- DDK：

  ```
  gpg --verify "MSpore_DDK****tar.gz.asc"
  ```

- 返回信息中**“27A74824”**为公钥ID。

- 提示信息返回**“Good signature”**且信息中无WARNING或FAIL，表明此签名为有效签名，软件包完整性校验通过。

- 若提示信息存在WARNING或FAIL，则表明验证不通过，请参见[软件包完整性校验返回WARNING或 FAIL](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0139382250.html)处理建议解决。

#### 3.4 解压安装包

​	使用安装MindSpore Studio的普通用户执行如下命令，解压**“mini_mindspore_studio_Ubuntu.rar”**安装包。

```
unzip mini_mindspore_studio_Ubuntu.rar
```

​	解压后包的内容：

- MindSpore Studio工具安装包：

- MindSpore-Studio_Ubuntu-x86-64.tar     Ubuntu操作系统x86架构下的MindSpore Studio安装包

- install.sh                                                   安装脚本

- check_sha.sh                                           校验以上文件的完整性，install.sh执行过程中会自动调用该脚本进行完整性

  ​                                                                 校验

- add_sudo.sh                                            为普通用户加权脚本

- del_sudo.sh                                              删除普通用户权限脚本

- profiling_sudo.sh                                      profiling加权脚本

​	安装MindSpore Studio时，安装脚本会自动加载DDK安装包中的相关内容，所以无需解压DDK安装包。

### 4. MindSpore Studio安装

- 当前不支持在一台机器上安装多个MindSpore Studio，如果系统上已存在MindSpore Studio，请先参见[卸载MindSpore Studio](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0136384622.html)将原Studio卸载后再执行安装，需要使用MindSpore Studio的安装用户进行卸载。
- 卸载MindSpore Studio后，在重新安装前，请先切换到root用户清空“/tmp”、“/dev/shm”目录，防止后续切换到普通用户安装时存在权限不足的问题。

#### 4.1 安装

##### （1）引导安装

​	引导安装方式会引导用户进行参数配置，并且同时安装MindSpore Studio和DDK，推荐用户优先使用这种安装方式。操	作步骤如下：

1. 切换到root用户为普通用户加权。

   ```
   su root
   cd /home/username/director
   ./add_sudo.sh username
   ```

2. 切换到普通用户执行**./install.sh** 或 **bash install.sh**安装脚本。

   安装MindSpore Studio时，脚本会自动加载DDK安装包中的相关内容，完成DDK的安装，DDK的默认安装路径为$HOME/tools/che/ddk。

3. 按照引导完成配置。

   a. 提示用户是否安装Linux内核补丁：

   ​	[INFO] Please make sure that the Linux Kernel Version is higher than 4.18 or apply this patch

   ​	 https://bugzilla.kernel.org/attachment.cgi?id=277305 when Linux Kernel Version below 4.18, otherwise maybe    	stuck when converting model using omg with TE plugin. Continue? [Y/N]（输入Y/y回车不安装补丁，继续进行工	具安装；输入N/n回车退出安装，由用户先安装内核补丁）

   b. 选择是否使用默认配置进行安装：

   ​	[INFO] Installation will run automatically with DEFAULT config, [Y/N]: （输入Y/y回车会按默认配置进行安装，输入	其他后续会让用户输入参数配置信息）

   c. 配置IP（MindSpore Studio 启动所用的IP地址），该操作分两种情况：

   ​	[INFO] Your ip address is xxx.xxx.xxx.xxx

   ​	1.[INFO] Press ENTER to continue or input a NEW ip address: 

   ​	（访问MindSpore Studio的IP地址，若环境有eth0的网卡，获取到了eth0的网卡ip，用户可以按回车确认或者输入		通过**ifconfig**命令查询出的有效ip，若有多张网卡，由用户自行选择）

   ​	2.[INFO] Please input your ip address:

   ​	（访问MindSpore Studio的IP地址，如果环境没有eth0网卡，未获取到eth0网卡的ip，用户必须输入通过**ifconfig**	  		命令查询出的有效ip）	

   d.（3.b输入Y/y回车选择默认配置，则无此步骤）配置MindSpore Studio监听端口。

   ​	[INFO] Please input you port (default is 8888): （输入port，或者按回车使用默认配置）

   e.（3.b输入Y/y回车选择默认配置，则无此步骤）配置MindSpore Studio运行期间相关工具路径。

   ​	[INFO] Please input tool path (default is /home/*username*/tools): （输入tool path，或者按回车使用默认配置）

   f.（3.b输入Y/y回车选择默认配置，则无此步骤）配置profiling监听端口。

   ​	[INFO] Please input you profiler port (default is 8099): （输入profiler_port，或者按回车使用默认配置）

   g.（3.b输入Y/y回车选择默认配置，则无此步骤）配置profiling后台apache服务用户名。

   ​	[INFO] Please input you apache_user (default is msvpUser): （输入apache_user，或者按回车使用默认配置）

   h. 选择备份路径（MindSpore Studio卸载时，用于数据保存的地址）：

   ​	[INFO] Please input backup path: (default is /xxx): （输入回车使用默认路径备份，xxx为默认路径地址，输入其他	会按照新的路径进行备份）

   i.（只有3.h选择的备份路径不为空，才会出现如下信息）是否从备份路径加载数据：

   ​	[INFO] Do you want to load data from *backup path*? [Y/N]: （输入Y/y回车加载备份路径下的数据，输入N/n不加	 	载）

   j.（用户安装目录不为空才会出现该信息，如果安装目录为空，则不出现该提示）是否需要备份tools安装目录：

   ​	[INFO] /home/*username*/tools is not empty. Files in the directory will be cleared during installation. Are you sure to 	back up them?[Y/N]: （输入Y/y回车退出安装，由用户先备份安装目录，再重新执行/.install.sh脚本进行安装；输入	N/n回车删除/home/*username*/tools下的内容，继续安装MindSpore Studio）

4. 安装MindSpore Studio和DDK。

   若出现**“Install successfully”**，则表明安装成功，MindSpore Studio会安装在“~/tools”目录。

   安装成功或安装失败，都会自动执行del_sudo.sh脚本，收回普通用户的权限。若安装失败后，用户想重新执行安装脚本，则需要重新执行[1](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0170977221.html#ZH-CN_TOPIC_0170977221__li18642181753310)进行加权操作。

   - 如果安装失败，请在“~/tools/log/mind_log”目录中查看相应日志文件，根据错误提示解决。
   - 如果提示profiling安装失败，请在“~/tools/log/profilerlog”目录中查看相应日志文件，根据错误提示解决。或参见[Profiling安装或启动失败](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0175384154.html)根据具体问题获取相关解决方法。

##### （2）手动安装（未实验）

​		请使用安装MindSpore Studio的普通用户执行如下操作。

1. 可选：解压安装包。执行如下命令解压[解压安装包](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0170705867.html)获得的MindSpore Studio安装包**“MindSpore-Studio_Ubuntu-x86-64.tar”**。

   ```
   tar -xvf MindSpore-Studio_Ubuntu-x86-64.tar
   ```

   

2. 切换到root用户为普通用户加权。（请在安装包所在目录*director*下，执行如下加权操作，执行完后再切换到普通用户）

   ```
   su root
   ./add_sudo.sh username
   ```

   如果不执行上述加权操作，则会在执行安装脚本时出现如下提示信息，停止安装。

   Please check if add_sudo.sh, del_sudo.sh exists and execute the add_sudo.sh script with root privileges

3. 切换到安装MindSpore Studio的普通用户配置文件。

   进入解压后的“scripts”目录，该目录下会包含一个**env.conf**配置文件，该文件为参数配置文件。

   配置文件中有ip、port、toolpath、backup、profiler_port、install_user、use_eth0、package_path参数可配置，各参数说明如下：

   - ip：

     MindSpore Studio启动所用的IP地址。该参数在安装的时候必须配置。

     若在此处配置成ip=any，则会在启动的时候去自动获取网卡ip，用于启动。

   - port：MindSpore Studio监听端口。可以自行修改，需配置为1024~65535的整数，并且请确保该端口不被占用。默认为8888。查询端口是否被占用的命令是：netstat -an | grep *端口号*

   - toolpath：运行期间相关工具的路径，如数据库路径db等。可以自行修改，默认为~/tools。

   - backup：存放project、my-model、my-datasets备份的路径。可以自行修改，默认为~/wsbackup。

   - profiler_port：profiling监听端口。可以自行修改，需配置为1024~65535的整数，并且请确保该端口不被占用。默认为8099。

     查询端口是否被占用的命令是：

     netstat -an | grep *端口号*

     - 必须保证端口未被占用，可执行**netstat -an | grep portnumber**查看。由于部分端口Chrome浏览器无法用于自定义（例如，6666、6000、6667等），建议将端口号设置为大于7000。
     - 必须保证这些端口未被占用：exec：4412、terminal：4411、wsagent_debug：4403、wsagent：4401、hiai-agent：3002、cce_profiler：3001、hiai_log：3003、profiler_port。

   - max_log_size：~/tools/log/hiai_agent_log文件夹下用户设置log文件最大个数，用户可自行配置，默认值为10个（考虑磁盘空间影响，该参数不建议配置太大）。

   - apache_user：profiling后台apache服务用户名。不支持修改。

   - install_user：代表当前安装的用户。该参数在安装的时候必须配置。

     普通用户安装，用户必须具有创建tool path和backup path的权限。

   - package_path：加权脚本add_sudo.sh所在路径，需配置为绝对路径，例如/home/username/director。

     - 该参数手动安装时必须配置，如果不配置，会导致profiling安装失败。
     - 如果卸载后重新安装MindSpore Studio或安装失败需要重新执行安装命令，该参数需要重新修改为加权脚本所在路径，例如/home/*username/director。*

   - use_eth0：配置ip为any时，可以选择是否默认用eth0启动。

     在安装的时候，use_eth0默认为true，意思为当配置ip为any时，选择默认用eth0启动；当use_eth0为false时，用户可以在多个网卡中选择输入ip并启动。

   - load_data：是否需要从backup备份路径加载数据，当配置为true，表示从备份路径加载数据，当配置为false时，表示不加载数据。默认为true。

4. 在安装MindSpore Studio的普通用户下执行安装脚本。

   在**scripts**目录下包含一个**mind_studio.sh**脚本，该文件为安装脚本，执行前必须完成[3](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0137313012.html#ZH-CN_TOPIC_0137313012__li18888165015306)。

   - 执行以下命令完成MindSpore Studio和DDK的安装，适用于正常场景。

     ```
     bash mind_studio.sh install {absolute_ddk_path}
     ```

   - 执行以下命令完成MindSpore Studio单独安装，适用于MindSpore Studio未正常安装的场景。

     ```
     bash mind_studio.sh install
     ```

   - 执行以下命令完成DDK单独安装，适用于DDK未正常安装的场景，在该场景下安装完DDK之后，需要重新启动

     MindSpore Studio。

     ```
     bash mind_studio.sh installDDK {absolute_ddk_path}
     ```

     {absolute_ddk_path}为DDK包的绝对路径，例如：/home/*username*/*MSpore_DDK****tar.gz*。

5. （用户安装目录不为空才会出现该信息，如果安装目录为空，则不出现该提示）是否需要备份tools安装目录：

   [INFO] /home/*username*/tools is not empty. Files in the directory will be cleared during installation. Are you sure to back up them? [Y/N]:（输入Y/y回车退出安装，由用户先备份/home/*username*/tools下的内容，再重新执行脚本安装；输入N/n删除/home/*username*/tools下的内容，继续安装MindSpore Studio）

##### （3）验证安装结果

1. 执行安装的流程

   MindSpore Studio安装会执行以下流程：		

   1. 安装mongodb数据库。
   2. 安装DDK。
   3. 安装HiAI CCE Profiler性能分析工具服务。
   4. 启动MindSpore Studio服务。
   5. 检测是否需要导入备份数据（backup路径下的数据）。
   6. 安装apache服务和PHP，并且启动apache服务。

   到这一步，如果您已经安装成功，就可以开始使用MindSpore Studio进行开发了。

2.安装成功检查

- 安装完成后通过Chrome浏览器访问如下网页地址，查看能否访问MindSpore Studio界面，能够访问成功说明

  MindSpore Studio工具安装成功，否则说明安装失败。

  ```
  https://IP:PORT   //如本机https://192.168.2.110:8888
  ```

  如果通过上述地址无法访问MindSpore Studio界面，请参见[MindSpore Studio安装完成后通过Chrome浏览器无法访问界面](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0178442393.html)解决。

- 通过Chrome浏览器访问如下网页地址，查看能否访问Profiling界面，能够访问成功说明Profiling工具安装成功，否则说明安装失败 。

  ```
  https://IP:Profiler_port             
  ```

- 查看版本安装是否正确，详细操作方法请参见[查询MindSpore Studio版本](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0141638662.html)。

- 在安装MindSpore Studio后，为保证安全，建议修改MongoDB数据库用户的默认密码，修改方法请参见[修改用户密码](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0143676890.html)。

- IP为MindSpore Studio安装服务器的IP，MindSpore Studio默认端口为8888，Profiling默认端口为8099，如果IP、Port以及Profiler_port为映射之后的，则需填写映射之后的IP和端口信息，您可以在“~/tools/scripts/env.conf”文件中修改IP和端口信息。

  \- “~/tools”是默认的toolpath路径，该路径可在安装MindSpore Studio时由用户自定义，您可以在“scripts/env.conf”文件通过toolpath参数查看实际路径。

- 登录MindSpore Studio界面的用户名默认为**“MindSporeStudioAdmin”**，不支持修改和新建。初始密码为**“Huawei123@”**，请参见[修改密码](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0146196781.html)。

- 登录Profiling界面的用户名为**msvpadmin**，初始密码为**Admin12#$**，请参见[展示性能分析数据](https://ascend.huawei.com/documentation/details/zh/v1.1.1.1/f4f39edd90e011e9a97afa163e714aa5/zh-cn_topic_0141811166.html)章节创建普通用户。

3. 版本相关说明

- 安装MindSpore Studio时需要设置两个路径：
  - toolpath：数据集、model zoo的存储位置。
  - backup：卸载MindSpore Studio时备份工程目录时需要存放的目录。
- 创建的工程存储在“/projects”目录下，该目录暂时不支持更改。

#### 4.2 常用操作

##### （1）启动MindSpore Studio

​	请在安装MindSpore Studio普通用户下执行如下操作。

​	在linux系统的“~/tools/bin”目录下执行如下命令启动MindSpore Studio：

​	**bash start.sh**

​	执行该命令将运行如下程序和服务：

- mongodb服务，对应进程查看方法：**ps -ef | grep mongod**
- HiAI_CCE-Profiler性能分析工具服务，对应进程查看方法：**ps -ef|grep httpd**、**ps -ef|grep redis**。
- MindSpore Studio服务，对应进程查看方法：**ps -ef | grep java | grep -v \'grep\' | grep HiAI-Studio-.\*/tomcat/bin/bootstrap.jar**

​	脚本执行完成后通过Chrome浏览器访问如下网页地址，查看能否访问MindSpore Studio界面，能够访问成功说明	 	 	MindSpore Studio启动成功，否则说明启动失败。

```
https://IP:Port
```

​	IP为MindSpore Studio安装服务器的IP，MindSpore Studio默认端口为8888，如果IP以及Port为映射之后的，则需填写	映射之后的IP和端口信息，您可以在“~/tools/scripts/env.conf”文件中修改IP和端口信息）。

##### （2）停止MindSpore Studio

​	请在安装MindSpore Studio普通用户下执行如下操作。

​	在Linux系统的“~/tools/bin”目录下执行如下命令停止MindSpore StudioMindSpore Studio。

```
bash stop.sh
```

​	执行该命令后将会停止如下程序和服务：

- mongodb服务

- HiAI_CCE-Profiler性能分析工具服务

- MindSpore Studio服务

  停止后，Chrome浏览器不能再访问MindSpore Studio界面。

##### （3）修改IP地址

​	若用户想要更换Ubuntu服务器的IP地址，则MindSpore Studio安装使用的IP地址也要随之更换，方法如下：

- 如果env.conf文件中的IP配置为Ubuntu服务器IP地址，则修改IP时，直接将env.conf文件中的IP地址改为新的Ubuntu服务器IP。

- 如果env.conf文件中的IP配置为any：

  - 如果env.conf文件中use_eth0取值为true，则修改eth0的IP地址，重新启动MindSpore Studio，新的IP地址生效。

  - 如果env.conf文件中use_eth0取值为false，则重新启动MindSpore Studio，在多个网卡中选择输入IP，新IP地址生效

    注：env.conf文件路径：~/tools/scripts/env.conf。

#### 4.3 版本升级

​	本服务器安装的版本为1.1.T22.B883，不支持在线升级。只能卸载后重新安装。



## 三，Atlas 200 DK部署

### 1. 简介

​	本文描述了用户在使用Atlas 200 Developer Kit（Atlas 200 DK）运行AI应用程序前的准备工作，包括系统SD卡的制	 	作，通过MindSpore Studio进行Atlas 200 Developer Kit开发者板基本信息的管理以及MindSpore Studio中交叉编译环	境的配置等。

### 2. 单板使用注意事项

- 上电前一定检查确保Atlas 200 AI加速模块正确扣在底板上，否则无法正常开机。
- 若需要使用摄像头功能或者内部接口时，需要拆卸上盖，请在断电情况下拆卸上盖。
- 请用户在使用非标配的电源适配器时，请注意供电范围及供电功率满足板卡要求。
- 信号电平为3.3V，接口使用时一定注意电平的匹配，否则会造成单板损坏。
- 40-pin扩展插针未进行严格的静电防护设计，请注意预防静电及不要带电插拔

### 3. 使用流程

![](/home/winston/图片/atlas_200_dk/使用流程.png)

​	注：本流程是发货时无SD卡的使用流程，如果发货时带SD卡，请从Atlas 200 DK开发者板上电开始使用。

### 4. 配置Atlas 200 DK

#### 4.1 连接Atlas 200 DK开发者板与MindSpore Studio

​	Atlas 200 DK开发者板支持通过USB接口或者网线与MindSpore Studio进行连接，其中Atlas 200 DK开发者板的IP规划   	如下：

- USB网卡的默认IP地址为：192.168.1.2
- NIC网卡的默认IP地址为：192.168.0.2

​	MindSpore Studio若想与Atlas 200 DK开发者板通信，需要配置与Altas 200 DK开发者板在同一网段的IP地址。

#### 4.2 操作步骤

1. 将MindSpore Studio与Atlas 200 DK开发者板连接。

   有以下两种连接方式：

   - 通过USB端口与Atlas 200 DK开发者板连接，请参考2。
   - 使用网线通过路由器或者交换机与Atlas 200 DK 开发者板连接，请参考3。

2. 通过USB连接场景下配置MindSpore Studio所在服务器的IP地址。

   若MindSpore Studio所在服务器通过USB端口与Atlas 200 DK开发者板直连，则修改该服务器的地址为192.168.1.x（x取值范围为0~1，3~255）。通过USB连接时，Atlas 200 DK开发者板的默认地址为192.168.1.2，支持USB2.0和USB3.0。

​	通过USB端口连接Atlas 200 DK开发者板时，需要配置USB静态IP，下面提供通过脚本配置与手工配置两种方法：

- 通过脚本配置

  a. 从https://github.com/Ascend/tools/blob/master/configure_usb_ethernet.sh下载configure_usb_ethernet.sh到	 	 	MindSpore Studio所在Ubuntu服务器的任一目录，例如*/home/ascend/config_usb_ip/*

  - 若从github上下载单个文件，请进入文件后，右键单击Raw，选择“链接另存为”
  - 通过脚本配置仅针对首次配置USB网卡对应IP地址的场景。USB网卡IP已经配置，若需要修改其IP地址，请参考手工配置修改USB网卡的IP地址。

  b. 以root用户进入配置USB IP地址的脚本所在目录，例如*/home/ascend/config_usb_ip*

  c. 执行如下命令bash configure_usb_ethernet.sh -s ip_address进行USB IP地址的配置。

  ​	以指定的IP地址配置系统中USB网卡的静态IP地址，如果直接执行**bash configure_usb_ethernet.sh**，则以默认IP 	地址“192.168.1.166”进行配置。

  ​    注：如果有多个USB网卡，执行以下命令：**bash configure_usb_ethernet.sh -s** *usb_nic_name* *ip_address*

- 手工配置

  a. 以MindSpore Studio安装用户登录MindSpore Studio所在服务器，执行如下命令切换到root用户。

```
su  root
```

​	  b. 获取USB网卡名

```
ifconfig -a
```

​	  若系统中有多个USB网卡，可以通过拔插开发者板进行判定。

​	  c. 在“/etc/network/interfaces”文件中添加USB网卡的静态IP。执行如下命令打开interfaces文件：

```
vi /etc/network/interfaces
```

​	  配置interfaces文件，例如USB网卡名为 ，配置静态IP为*192.168.1.223*

```
auto enp0s20f0u4
iface enp0s20f0u4 inet static
address 192.168.1.223
netmask 255.255.255.0
```

​	  d. 修改NetworkManager.conf文件，避免重启后网络配置失效。执行如下命令打开“NetworkManager.conf”文件。

​		**vi /etc/NetworkManager/NetworkManager.conf**

​	  修改文件中的**“managed=false”**为**“managed=true”** 。

​	  e. 配置静态IP生效，执行以下命令：

```
ifdown enp0s20f0u4
ifup enp0s20f0u4
service NetworkManager restart
```

3. 通过网线连接场景下配置MindSpore Studio所在服务器的IP地址。

   若MindSpore Studio所在服务器使用网线通过路由器或者交换机与Atlas 200 DK开发者板直连，则修改该服务器地址为192.168.0.x（x取值范围为0~1，3~255）。

   通过网线连接时，Atlas 200 DK开发者板的默认地址为192.168.0.2，子网掩码为24位。

   配置方法如下：

- 以MindSpore Studio安装用户登录MindSpore Studio所在服务器，执行如下命令切换到root用户。

  ```
  su - root
  ```

- 在“/etc/network/interfaces”文件中添加虚拟的静态IP。执行如下命令打开interfaces文件：

  ```
  vi /etc/network/interfaces
  ```

  配置interfaces文件，例如添加一个  的静态IP为*192.168.0.223*

  ```
  auto eth0:1
  iface eth0:1 inet static
  address 192.168.0.223
  netmask 255.255.255.0
  ```

- 修改“NetworkManager.conf”文件，避免重启后网络配置失效。执行如下命令打开“NetworkManager.conf”文件。

  **vi /etc/NetworkManager/NetworkManager.conf**，修改文件中的**“managed=false”**为**“managed=true”** 。

- 重启网络相关服务

  ```
  service network-manager restart
  ```

#### 4.3 密码修改

​	HwHiAiUser用户为通过MindSpore Studio制作SD卡时创建的默认用户，此用户的默认密码是**Mind@123**。Atlas 200 	 	DK开发者板与MindSpore Studio连接成功后需要修改Altlas 200 DK开发者板中HwHiAiUser用户的初始密码。

1. 在MindSpore Studio服务器中以HwHiAiUser用户ssh登录到Atlas 200 DK开发者板。用户HwHiAiUser缺省登录密码为**Mind@123**。

2. 执行**passwd**命令修改HwHiAiUser密码。

   注：按照同样的操作也可以修改root用户的密码，只是需要执行su root进入rott用户。

#### 4.4 MindSpore Studio中添加Atlas 200 DK开发者板

​	MindSpore Studio提供Atlas 200 DK开发者板配置管理功能，实现对Atlas 200 DK开发者板的设备连接状态检测及设备	信息修改功能。如实现添加用户，连接用户，修改开发板ip地址（只支持网口连接）。

#### 4.5 版本升级

​	可在线升级，也可以手动升级。

##### 	1. 手动升级

1. 将Atlas 200 DK开发者板升级包“mini_developerkit-xxx.rar”以MindSpore Studio安装用户上传到MindSpore Studio服务器任一目录下，并进入此目录。

2. 以Mind Studio安装用户进入存放开发者板升级包的目录，并在当前目录下以HwHiAiUser用户SSH登录到开发者板，然后切换到root用户。

   **ssh HwHiAiUser@192.168.1.2**

   **su - root**

​	3. 将开发者板升级包“mini_developerkit-xxx.rar”拷贝到开发者板的“/opt/mini”目录下。

   	**cd /opt/mini**

   	**sftp username@192.168.1.223**

   	sftp>**cd /home/ubuntu/software**

   	sftp> **get mini_developerkit-xxx.rar**

​       sftp> **exit**

​	4. 执行以下命令进行升级前准备。

​	    **./minirc_install_phase1.sh**

​	5. 执行重启命令，进行开发者板的重启，从而完成升级。

​	    **reboot**

​	注意：升级过程中请勿将Atlas 200 DK开发者板断电，升级时间15分钟左右。

##### 	2. 升级后检查

- 在MindSpore Studio服务器中以HwHiAiUser用户ssh登录到Atlas 200 DK开发者板。``

  **ssh HwHiAiUser@192.168.1.2**

- 执行如下命令查看开发者板的Ascend 310软件升级后的版本号。

  **cat /etc/sys_version.conf**

- 执行如下命令查看升级日志。

  **cd /var/davinci/log**

  **cat upgrade.log**

  **cat firmware_upgrade_progress.log**

- 查看升级日志是否有错误信息，如果无错误信息，代表升级成功。

  如果有错误信息，请通过MindSpore Studio前台界面连接开发者板查看日志进行定位



## 四，应用开发

### 1. 概述

MindSpore Studio的Matrix流程编排功能提供AI引擎可视化拖拽式编程及算法代码自动生成技术，Matrix是一个通用的业务流程引擎，主要包含Matrix Agent与Matrix Daemon。

- **Matrix Agent**：运行在**Host侧**（**Host侧**指与Device相连接的X86服务器、WindowsPC、3559以及AMD服务器等），会利用Device提供的NN计算能力，完成业务功能。

- **Matrix** **Daemon**：运行在**Device侧**（**Device侧**指安装了NPU芯片的硬件设备，利用PCIe接口与Host侧连接），为Host提供NN计算能力。

### 2.业务编排

- 创建Mind类型工程：Target可选择为ASIC、Atlas DK和Local三种。
- 编排流程：模型文件类型可选择为分类网络，检测网络，不带预处理节点的流程编排。
- 工程编译：生成对应的源码和执行脚本。
- 工程运行：执行编译后的脚本输出结果。
- 运行结果查看：运行完成之后会在out文件夹下生成结果文件夹，可在Postprocess类型节点上查看相应结果。结果分三类：Image Result、Statistical Result和Profiling Result。

编排流程可见下图：

![](/home/winston/图片/atlas_200_dk/编排流程.png)

### 3.Atlas场景软件架构

Atlas 200 DK场景Matrix逻辑架构如图：

![](/home/winston/图片/atlas_200_dk/Atlas场景软件架构.png)



Matrix为通用业务流程执行引擎，主要提供以下功能：

- 提供Ascend 310芯片功能接口。
- 完成与APP进行控制命令和处理数据的交互。
- 根据配置文件完成业务流程的建立。
- 根据命令完成业务流程的销毁及资源回收。
- Engine调用DVPP的API接口实现媒体预处理。
- Engine调用模型管家（AIModelManger）的API接口实现模型推理。

DVPP Executor（Digital Vision Pre-Processing Executor），提供API接口

- 实现视频解码、视频编码、JPEG编解码、PNG解码、视觉预处理（包括裁剪、缩放等）功能。

Framework：离线模型框架，主要是先功能：

- 离线模型转换：可以将caffe、tensorflow等开源框架模型转换成Ascend 310芯片支持的Davinci模型。
- 离线模型执行：模型管家（AIModelManger）提供API接口，用于离线模型加载和推理功能。
- 支持自定义算子：对于Ascend 310不支持的算子，用户可以自定义实现。

CCE：CCE（cube-based computing engine）加速库通过API的方式，为上层应用（Framework或者Matrix）提供加速。

Runtime：运行于APP进程空间，为APP提供了Ascend 310设备的Memory管理、Device管理、Stream管理、Event管理、Kernel执行等功能。

### 4.应用场景

- 单APP，单线程：一个APP启动一个线程，由该线程发起推理任务。
- 单APP，多线程：一个APP启动多个线程，由线程各自发起推理任务。
- 多APP，每个APP单线程：多APP各自启动一个线程，由线程各自发起推理任务。
- 多APP，每个APP多线程：多APP各自启动多个线程，由线程各自发起推理任务。

## 五，FAQ

### 1. MindSpore Studio部署问题

#### 1.1 安装过程中要求内核版本>=4.18，本服务器内核版本为4.15,需要升级

​	   文档中提供的是打升级补丁。实际安装过程中是重新安装新的内核，并切换内核，具体教程如下链接：

​		https://timelate.com/archives/update-the-latest-kernel-of-ubuntu.html

#### 1.2 依赖项出现问题，在安装python和jdk时出现依赖项未安装，需要libssl版本>=1.1.

​		这是因为Ubuntu16.04依赖库版本不支持，需要自己下载libssl1.1及以上版本。

​		下载链接：https://pkgs.org/download/libssl1.1

#### 1.3 在验证的时候总是出现验证失败，输出信息如下：

```
	gpg: 假定被签名的数据是‘mini_mindspore_studio_Ubuntu.rar’
	gpg: 于 2019年06月26日 星期三 19时31分47秒 CST 创建的签名，使用 RSA，钥匙号 27A74824
	gpg: 已损坏的签名，来自于“OpenPGP signature key for Huawei software (created on 30th 		 	 Dec,2013) <support@huawei.com>”
```

​		  解决方案给的是重新下载软件包，但是下载之后依旧是这样，下载tool是错误的。

​		  要在具体的软件包目录下去下载文件，不然会出错。最终解决方案是重新下载完整软件包。



### 2. Atlas 200 DK部署问题

#### 2.1 用网口连接MindSpore Studio所在服务器和Atlas 200 DK，连接成功，在MindSpore Studio上面添加开	发板之后修改了其ip地址，然后删除设备。在这之后就无法通过网口连接方式添加开发板，同时也不能通	过ssh登录HwHiAiUser用户，但是利用USB连接方式则可以添加也可以登录。

​	原因是修改之后的ip地址和连网线的ip不处于同一个网段，不能互相通信造成的。

​	解决办法：手动修改Ubuntu服务器的ip地址（192.168.0.100），将其改为和开发板ip（192.168.2.100）处于同一网段	的ip地址（192.168.2.226），重启网络后就可以正常的运行了。在MindSpore Studio添加开发板之后将其ip修改回原来	的地址，同时将Ubuntu服务器的地址也改回来。同样可以正常运行。

### 3. 例程运行问题

#### 	3.1运行Resnet编译成功，run成功，但是无法查看分类结果，输出如下：

![image result](/home/winston/图片/atlas_200_dk/iamge result fail.png)

![statistical result fail](/home/winston/图片/atlas_200_dk/statistical result fail.png)

​	这是因为MIndSpore Studio和atlas 200dk开发板版本不一样导致的结果，需要进行升级到同一版本。升级操作看版本升	级介绍。