### VIO代码修改

VIO过程中关于参数设置的问题
VO不能再显示器上显示可能是：

1. 因为开发板的输出和显示器的帧频不一致。要统一设置为60帧，需要进入VO部分函数修改。
2. 输入尺寸可能存在问题，需要width=1820，height=1080
3. 代码整体运行步骤：
   假设输入为 ./sample_vio 0 0
   main()
   	SAMPLE_VIO_8K30_PARALLEL()    //进入parallel SDR8     VI - VPSS - VO - HDMI模式
   		//step 1 得到所有的传感器信息，最终需要一个VI，一个mipi
   		SAMPLE_COMM_VI_GetSensorInfo()  //获取传感器信息
   		SAMPLE_COMM_VI_GetComboDevBySensor()  //根据传感器信息获取mipi设备信息
   		//step 2 得到输入的size
   		SAMPLE_COMM_VI_GetSizeBySensor()//不同的传感器获取的图像enSize不同
   		SAMPLE_COMM_SYS_GetPicSize()    //根据enSize得到相应的picSize
   		//step 3 初始化SYS（MPP）和 common VB （公共视频缓存池）
   		COMMON_GetPicBufferSize()       //得到图像缓存区的size
   		VI_GetRawBufferSize()           //得到，raw格式图像的缓存区size
   		//SYS初始化
   		SAMPLE_COMM_SYS_Init()          
   			HI_MPI_SYS_Exit ()          //去初始化MPP系统
   			HI_MPI_VB_Exit()              //去初始化MPP视频缓存池
   			HI_MPI_VB_SetConfig()       //设置MPP视频缓存池属性
   			HI_MPI_VB_Init()              //初始化MPP视频缓存池   
   			HI_MPI_SYS_Init()           //初始化MPP系统
   		//VI，VPSS 工作模式参数设置
   		SAMPLE_COMM_VI_SetParam()       	
   				HI_MPI_SYS_GetVIVPSSMode()
   				HI_MPI_SYS_SetVIVPSSMode()
   		//step 4 开始VI
   		SAMPLE_COMM_VI_StartVi()
   			//初始化mipi
   			SAMPLE_COMM_VI_StartMIPI()
   				SAMPLE_COMM_VI_SetMipiHsMode()        //设置mipi模式
   				SAMPLE_COMM_VI_EnableMipiClock()      //打开输入mipi时钟
   					SAMPLE_COMM_VI_GetSnsInputMode()  
   				SAMPLE_COMM_VI_ResetMipi()            //重置mipi设置
   					SAMPLE_COMM_VI_GetSnsInputMode()
   				SAMPLE_COMM_VI_EnableSensorClock()    //使能传感器输入时钟
   				SAMPLE_COMM_VI_ResetSensor()              //重置传感器设置
   				SAMPLE_COMM_VI_SetMipiAttr()              //设置mipi属性参数
   					SAMPLE_COMM_VI_GetComboAttrBySns()
   				SAMPLE_COMM_VI_UnresetMipi()               
   				SAMPLE_COMM_VI_UnresetSensor()
   		//step 5 开始VPSS。需要一个grp（组）
   		SAMPLE_COMM_VPSS_Start()
   			//开始 VPSS grp
   			HI_MPI_VPSS_CreateGrp()                       //创建一个VPSS Group
   			HI_MPI_VPSS_SetChnAttr()                      //设置VPSS通道属性
   			HI_MPI_VPSS_EnableChn()                       //启用VPSS通道
   			HI_MPI_VPSS_StartGrp()                        //启用VPSS Group
   		//step 6  VI bind VPSS, for total parallel, no need bind
   				  在其他的模式中需要绑定
   		//step 7  启用VO
   		SAMPLE_COMM_VO_GetDefConfig()                     //得到输出设备配置
   		SAMPLE_COMM_VO_StartVO()
   			//Set and start VO device VoDev
   			SAMPLE_COMM_VO_StartDev()
   				HI_MPI_VO_SetPubAttr()                    //设置视频输出设备的公共属性
   				HI_MPI_VO_Enable()                        //启用视频输出设备
   			//Set and start layer VoDev
   			SAMPLE_COMM_VO_GetWH()                        //得到输出的宽高
   			//Set display rectangle if changed
   			if 
   				HI_MPI_VO_SetVideoLayerPartitionMode()    //设置视频层的分割模式
   			if
   				HI_MPI_VO_SetDisplayBufLen()              //设置视频层上的显示缓存长度
   			SAMPLE_COMM_VO_StartLayer()              
   				HI_MPI_VO_SetVideoLayerAttr()             //设置视频层属性
   				HI_MPI_VO_EnableVideoLayer()              //使能视频层
   			if
   				SAMPLE_COMM_VO_StopDev()                  //禁用设备
   			if
   				HI_MPI_VO_GetVideoLayerCSC()              //获取视频层CSC
   				if
   					SAMPLE_COMM_VO_StopDev()              //禁用设备
   				HI_MPI_VO_SetVideoLayerCSC()              //设置视频层CSC
   				SAMPLE_COMM_VO_StopDev()                  //禁用设备
   			//start vo channels
   			SAMPLE_COMM_VO_StartChn()
   				HI_MPI_VO_GetVideoLayerAttr()             //获取视频层属性
   				HI_MPI_VO_SetChnAttr()                    //设置指定视频输出通道的属性
   				HI_MPI_VO_EnableChn()                     //启用指定的视频输出通道
   			if
   				SAMPLE_COMM_VO_StopLayer()                //禁用视频层
   				SAMPLE_COMM_VO_StopDev()                  //禁用视频输出设备
   			//Start hdmi device
   			SAMPLE_COMM_VO_HdmiStartByDyRg()
   				HI_MPI_HDMI_Init()                        //初始化HDMI
   				HI_MPI_HDMI_Open()	                    //打开HDMI
   				HI_MPI_HDMI_GetAttr()                     //获取HDMI属性
   				HI_MPI_HDMI_SetAttr()                     //设置HDMI属性
   				HI_MPI_HDMI_Start()                       //启动HDMI输出
   			//Start mipi_tx device
   			SAMPLE_COMM_VO_StartMipiTx()                  //启用Mipi Tx
   		//VO bind VPSS
   		SAMPLE_COMM_VPSS_Bind_VO()
   			HI_MPI_SYS_Bind()                             //数据源到数据接收者绑定
   		SAMPLE_COMM_VPSS_UnBind_VO()                      //解绑
   		SAMPLE_COMM_VO_StopVO()                           //禁用VO
   		SAMPLE_COMM_VPSS_Stop()                           //禁用VPSS
   		SAMPLE_COMM_VI_StopVi()                           //禁用VI
   		SAMPLE_COMM_SYS_Exit()                            //MPP退出

### 

