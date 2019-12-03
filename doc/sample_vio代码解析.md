## sample_vio代码解析

```C
//mpp/sample/vio/sample_vio.c
int main(int argc, char* argv[])
{
    VO_INTF_TYPE_E enVoIntfType = VO_INTF_HDMI;
    
    if ((argc > 2) && (1 == atoi(argv[2])))
    {
        enVoIntfType = VO_INTF_BT1120;
    }

    s32Index = atoi(argv[1]);
    switch (s32Index)
    {
        case 0:
            s32Ret = SAMPLE_VIO_8K30_PARALLEL(enVoIntfType);
            break;
        ....
    }
}
```

因以`./sample_vio 0 0`的方式执行，故实际上会调用`SAMPLE_VIO_8K30_PARALLEL(VO_INTF_HDMI)`

```C
//mpp/sample/vio/sample_vio.c
HI_S32 SAMPLE_VIO_8K30_PARALLEL(VO_INTF_TYPE_E enVoIntfType)
{
    /************************************************
    step1:  Get all sensors information
    *************************************************/
    SAMPLE_COMM_VI_GetSensorInfo(&stViConfig);
    //mpp/sample/common/sample_comm_vi.c
    //最多接8个相机，各相机的类型由mpp/sample/Makefile.param中定义的SENSORx_TYPE确定
    
    /************************************************
    step 4: start VI
    *************************************************/
    s32Ret = SAMPLE_COMM_VI_StartVi(&stViConfig);
}
```



```C
//mpp/sample/common/sample_comm_vi.c
HI_S32 SAMPLE_COMM_VI_StartVi(SAMPLE_VI_CONFIG_S* pstViConfig)
{
    s32Ret = SAMPLE_COMM_VI_CreateIsp(pstViConfig);
}

HI_S32 SAMPLE_COMM_VI_CreateIsp(SAMPLE_VI_CONFIG_S* pstViConfig)
{
    for (i = 0; i < pstViConfig->s32WorkingViNum; i++)
    {
        s32ViNum  = pstViConfig->as32WorkingViId[i];
        pstViInfo = &pstViConfig->astViInfo[s32ViNum];
        
        s32Ret = SAMPLE_COMM_VI_StartIsp(pstViInfo);
    }
}

HI_S32 SAMPLE_COMM_VI_StartIsp(SAMPLE_VI_INFO_S* pstViInfo)
{
    for (i = 0; i < WDR_MAX_PIPE_NUM; i++)
    {
        if (pstViInfo->stPipeInfo.aPipe[i] >= 0  &&
            pstViInfo->stPipeInfo.aPipe[i] < VI_MAX_PIPE_NUM)
        {
            ViPipe      = pstViInfo->stPipeInfo.aPipe[i];
            u32SnsId    = pstViInfo->stSnsInfo.s32SnsId;
            
            s32Ret = SAMPLE_COMM_ISP_Sensor_Regiter_callback(ViPipe, u32SnsId);
            
            s32Ret = HI_MPI_ISP_Init(ViPipe);
        }
    }
}
```



```C
//mpp/sample/common/sample_comm_isp.c
HI_S32 SAMPLE_COMM_ISP_Sensor_Regiter_callback(ISP_DEV IspDev, HI_U32 u32SnsId)
{
    const ISP_SNS_OBJ_S* pstSnsObj;
    
    pstSnsObj = SAMPLE_COMM_ISP_GetSnsObj(u32SnsId);
    
    if (pstSnsObj->pfnRegisterCallback != HI_NULL)
    {
        s32Ret = pstSnsObj->pfnRegisterCallback(IspDev, &stAeLib, &stAwbLib);
    }
}

ISP_SNS_OBJ_S* SAMPLE_COMM_ISP_GetSnsObj(HI_U32 u32SnsId)
{
    SAMPLE_SNS_TYPE_E enSnsType;

    enSnsType = g_enSnsType[u32SnsId];
    switch (enSnsType)
    {
        case SONY_IMX334_MIPI_8M_30FPS_12BIT:
        case SONY_IMX334_MIPI_8M_30FPS_12BIT_WDR2TO1:
	    	return &stSnsImx334Obj;
    }
}
```



```C
//mpp/component/isp/user/sensor/hi3559av100/sony_imx334/imx334_cmos.c
ISP_SNS_OBJ_S stSnsImx334Obj =
{
    .pfnRegisterCallback    = sensor_register_callback,
    .pfnUnRegisterCallback  = sensor_unregister_callback,
    .pfnStandby             = imx334_standby,
    .pfnRestart             = imx334_restart,
    .pfnMirrorFlip          = HI_NULL,
    .pfnWriteReg            = imx334_write_register,
    .pfnReadReg             = imx334_read_register,
    .pfnSetBusInfo          = imx334_set_bus_info,
    .pfnSetInit             = sensor_set_init
};

static HI_S32 cmos_init_sensor_exp_function(ISP_SENSOR_EXP_FUNC_S *pstSensorExpFunc)
{
    CMOS_CHECK_POINTER(pstSensorExpFunc);

    memset(pstSensorExpFunc, 0, sizeof(ISP_SENSOR_EXP_FUNC_S));

    pstSensorExpFunc->pfn_cmos_sensor_init = imx334_init;
    pstSensorExpFunc->pfn_cmos_sensor_exit = imx334_exit;
    pstSensorExpFunc->pfn_cmos_sensor_global_init = sensor_global_init;
    pstSensorExpFunc->pfn_cmos_set_image_mode = cmos_set_image_mode;
    pstSensorExpFunc->pfn_cmos_set_wdr_mode = cmos_set_wdr_mode;
    pstSensorExpFunc->pfn_cmos_get_isp_default = cmos_get_isp_default;
    pstSensorExpFunc->pfn_cmos_get_isp_black_level = cmos_get_isp_black_level;
    pstSensorExpFunc->pfn_cmos_set_pixel_detect = cmos_set_pixel_detect;
    pstSensorExpFunc->pfn_cmos_get_sns_reg_info = cmos_get_sns_regs_info;

    return HI_SUCCESS;
}

static HI_S32 sensor_register_callback(VI_PIPE ViPipe, ALG_LIB_S *pstAeLib,
                                       ALG_LIB_S *pstAwbLib)
{
    s32Ret  = cmos_init_sensor_exp_function(&stIspRegister.stSnsExp);
}
```

相机初始化函数的调用过程：

```C
//mpp/component/isp/user/firmware/src/main/mpi_isp_entry.c
HI_S32  HI_MPI_ISP_Init(VI_PIPE ViPipe)
{
    ISP_GET_CTX(ViPipe, pstIspCtx);
    
    if(HI_FALSE == pstIspCtx->bISPYuvMode)
    {
        s32Ret = HI_ISP_Init(ViPipe);
    }
}

HI_S32 HI_ISP_Init(VI_PIPE ViPipe)
{
    s32Ret = ISP_SensorInit(ViPipe);
}

//mpp/component/isp/user/firmware/src/main/isp_sensor.c
HI_S32 ISP_SensorInit(VI_PIPE ViPipe)
{
    SENSOR_GET_CTX(ViPipe, pstSensor);
    
    if (HI_NULL != pstSensor->stRegister.stSnsExp.pfn_cmos_sensor_init)
    {
        pstSensor->stRegister.stSnsExp.pfn_cmos_sensor_init(ViPipe);
    }
}
```

i2c底层驱动

```C
//mpp/component/isp/user/sensor/hi3559av100/sony_imx334/imx334_sensor_ctl.c
void imx334_init(VI_PIPE ViPipe)
{
    /* 1. sensor i2c init */
    imx334_i2c_init(ViPipe);
}

int imx334_i2c_init(VI_PIPE ViPipe)
{
    snprintf(acDevFile, sizeof(acDevFile),  "/dev/i2c-%u", u8DevNum);
    
    g_fd[ViPipe] = open(acDevFile, O_RDWR, S_IRUSR | S_IWUSR);
    ret = ioctl(g_fd[ViPipe], I2C_SLAVE_FORCE, (imx334_i2c_addr >> 1));
}
```



```C
//osdrv/opensource/kernel/linux-4.9.y_multi-core/drivers/i2c/i2c-dev.c
static long i2cdev_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
    switch (cmd) {
        case I2C_SLAVE_FORCE:
    }
}

static const struct file_operations i2cdev_fops = {
	.owner		= THIS_MODULE,
	.llseek		= no_llseek,
	.read		= i2cdev_read,
	.write		= i2cdev_write,
	.unlocked_ioctl	= i2cdev_ioctl,
	.open		= i2cdev_open,
	.release	= i2cdev_release,
};

static int i2cdev_attach_adapter(struct device *dev, void *dummy)
{
    cdev_init(&i2c_dev->cdev, &i2cdev_fops);
}

static int __init i2c_dev_init(void)
{
    /* Bind to already existing adapters right away */
	i2c_for_each_dev(NULL, i2cdev_attach_adapter);
}

module_init(i2c_dev_init);
```



读相机的寄存器

```C
//osdrv/opensource/kernel/linux-4.9.y_multi-core/drivers/i2c/i2c-dev.c
static ssize_t i2cdev_read(struct file *file, char __user *buf,
                           size_t count, loff_t *offset)
{
#ifdef CONFIG_I2C_HISI
	if (client->flags & I2C_M_DMA)
		ret = i2c_master_recv(client, tmp,
				max_t(size_t, reg_width, count));
	else
		ret = i2c_master_recv(client, tmp,
				max_t(size_t, reg_width, data_width));
#else
	ret = i2c_master_recv(client, tmp, count);
#endif
}

//kernel/linux-4.9.y_multi-core/arch/arm64/configs/hi3559av100_arm64_big_little_defconfig
//未定义CONFIG_I2C_HISI
```

