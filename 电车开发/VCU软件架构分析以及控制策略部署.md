# VCU软件架构分析

## 工程文件架构

​	工程文件下的包含以下3个文件夹分别是，"driverlib"，"targetConfigs"，"Usercode"。其中”Usercode“是由用户主动创建的文件夹，其余为新建工程模板是自动生成的文件夹。

![image-20250902164039538](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250902164039538.png)

* **driverlib** C2000驱动库代码。

​						![image-20250902164115267](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250902164115267.png)

* **targetConfigs** 主要用于存储仿真器连接的配置文件。

![image-20250902164133285](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250902164133285.png)

* **Usercode** 包含所有实际应用代码以及BSP代码。

​						![image-20250902164158141](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250902164158141.png)

​	其中，ASW是应用程序软件，BSW是板级应用程序软件。分别包含了对于的.c和.h文件。

## main函数架构

​	main函数之前的定义了多达几十个的全局变量，主要用于一些计算电机扭矩，电量，功率，里程的一些数学计算参数，还有用于仪表盘显示的参数，用于进行控制策略判断的变量等等。

```c
union VEH_STATE VehicleSta;

uint16_t CurrentMaxDischarge;//动力电池允许最大放电电流  0.1A
uint16_t CurrentMaxCharge;//动力电池允许最大充电电流  0.1A
uint16_t LimitPowerCoffe;//降功系数 99.9--0.0%
uint16_t BatAllownPower;//电池允许最大功率 0.1KW
uint16_t VehicleAllownPower;//整车允许最大功率 0.1KW
int16  motorSpd;//1Rpm
int16  aimTorque1Nm;//1N.m目标扭矩 （或转速控转矩上限）
int16  aimSpeed1Rpm;//目标转速（或转矩控转速上限）
uint32_t ExternalCharacteristicTorque = 0;//电机扭矩外特性
int16 motorTempra = 0;//电机温度 1℃
int16 mcuTempra = 0;//MCU电控温度 1℃
int16 pumpMotorTempra = 0;//泵电机温度 1℃
int16 pumpIntverTempra = 0;//泵电机控制器温度 1℃
int16 dcTempra = 0; //dc控制器温度 1℃
int16 VehicleSpeed = 0;//车速  0.1Km/h
uint32_t TotalMileage = 0;//总里程 0.1Km
int32  EnergyConsumption = 0;//能耗 Kwh/Km
int16  mcuMotorSpd = 0;//主驱电机转速 Rpm/min
int16  mcuTorque = 0;//主驱扭矩 1Nm
uint16_t BrkTqr_Limit = 0;//主驱最大制动扭矩
int16  pumpMotorSpd = 0;//泵电机转速 Rpm/min
int16  pumpTorque = 0;//泵电机扭矩 Nm
int16  pumpAngle = 0;//泵转向角度--方向盘转向反馈角度
int16  pumpAngleSpd = 0;//泵转向角速度——方向盘转向反馈角速度
uint16_t dcBusVoltage=0;//DC母线电压 0.1V
uint16_t batBusVoltage=0;//动力电池电压 0.1V
int16_t  batBusCurrent=0;//动力电池电流 0.1A
uint16_t BMS_SOC;//1%  SOC
uint16_t mcuBusVoltage=0;//主驱母线电压 0.1V
uint16_t pumpBusVoltage=0;//油泵母线电压 0.1V
uint16_t SlotCardFlag = 0;//刷卡标志  1：通过 0：未通过
uint16_t disChgConnectSta = 0;//0:未连接  1：交流连接  2：直流连接
uint16_t StartSignalFlag = 0;//start成立标志
uint16_t VehReadySta = 0;
int32    pumpAimSpd = 0;//泵目标转速 1Rpm/min
uint32_t m_DetaTime = 0;
uint32_t RunTimeTotal = 0;//运行时间 0.1H
uint32_t mileageLittle =1;//小计里程
uint32_t mileageTotal = 1;//总里程
uint16_t mileageMonitor[4] = {0,0,0,0};
uint32_t PowerConsumptionTotal = 0;//总耗电量 KWH
uint16_t EnergyConsumptionTotal = 0;//总能耗 KWH/KM
uint16_t VehSpdMode;//车速模式  0：无效  1：高速  2：低速  3：微动
uint16_t Request_VIN_Flag; // 0:无请求  1：请求
uint16_t VIN_WriteFlag;//1:写，0：写完成
uint16_t eepromWriteFlag;//EEPROM写标志
uint16_t StateMachineFlag;
uint16_t StepTempFlag;
uint16_t RecTimeOut[7] = {0,0,0,0,0,0,0};
int16    VacuumPressure = 0;//真空泵压力值，0.1Kpa
uint32_t K1InitVal = 0;
uint32_t TtlMlg1Val = 0;
uint16_t RemMlg = 0;//剩余里程 0.1kM
uint16_t absActivationFlag = 0;//ABS激活标志，1:激活 0：不激活
//ASCII码    0:0x30  A:0x65      Z:0x5A
//LL3AHCJ47EA013922    76 76 51 65 72 67 74 52 55 69 65 48 49 51 57 50 50
// uint16_t vinCodeTable[9] = {0x004C,0x4C33,0x4148,0x434A,0x3437,0x4541,0x3031,0x3339,0x3232};//VIN码数组
uint16_t vinCodeTable[9] = {0, 0, 0, 0, 0, 0, 0, 0, 0};//每个Uint16含有两个字节，共9个元素，18个字节，VIN共17个字节

uint16_t FeedBackGear = 1;    //回馈档位
```



​	main首先进行时钟的配置，引脚的配置，外设的初始化还有进行中断的配置，**初始化功能逻辑参数**，然后使能看门狗（用于防止死机，死机会自动重启）。

> 初始化功能模块逻辑十分重要，属于VCU控制策略之一，（仪表盘的显示的参数需要存储在EEPROM中，当做一个外部存储器使用）上电时需要从EEPROM进行参数的读取，然后是初始化故障代码。

​	然后就是主循环，主循环使用的是类似状态机的一个模式，通过后台的定时器产生中断进行计时，超过固定时间间隔则产生相应的操作状态，对相应步骤的函数进行调用。属于前后台模式。

​	使用的是用户定义的中断函数为InitCPUTimer（），其处于main（）--> InterruptInit（）下。其与ST MCU中的周期为100us的系统滴答定时器功能一致。其中断函数如下。

```c
uint16_t CpuTimer0_InterruptCount = 0;
uint16_t OuttimeFlag2ms =0;
uint16_t Ticker2ms = 0;
__interrupt void cpuTimer0ISR(void)
{
    CpuTimer0_InterruptCount++;//定时中断标志，置1则为定时100us时间到
    Ticker2ms++;//2ms定时计数变量

    if(Ticker2ms>=20)
    {
        Ticker2ms = 0;
        OuttimeFlag2ms = 1;//2ms定时时间到
    }
    
    Interrupt_clearACKGroup(INTERRUPT_ACK_GROUP1);
}
```

​	在主循环中无条件执行即为100us定时到边执行的CANA及CANB通信报文接收功能的函数。

```c
void ExecutiveRoutine100us()
{
    if(CpuTimer0_InterruptCount>=1)//100us定时时间到
    {
        m_DetaTime++;//步进计数变量，计时为500us时即进入下一步

        eCanaReceivedMsgFunc();//CAN a接收报文

        eCanbReceivedMsgFunc();//CAN b接收报文

        CpuTimer0_InterruptCount = 0;
    }
}
```

​	下一个便是，每个500us定时到之后都会执行的函数，主要功能为快速的采样数据，以及处理后台数据，最后顺便喂一下看门狗。

```c
void ExecutiveRoutine05ms()
{
    SamplVoltageCacl();//AD采样处理

    ScopeMsgDeal();//后台数据处理

#ifdef _NON_DEBUG
    //喂狗
    ServiceDog();
#endif

}
```

​	然后是每2ms时间间隔轮询的任务函数A、B、C、D。每个任务都负责两到三个主要任务，函数内部都严禁出现强制延迟函数，除非是由底层硬件操控的。main函数主循环内容如下。

```c
for(;;)
	{
	    if (m_DetaTime >= 5) //0.5ms执行周期
	    {
	        m_DetaTime = 0;

	        ExecutiveRoutine05ms();

	        step++;

	        if(1==step)
	        {
	            ExecutiveRoutine05ms_A();
	        }
	        else if(2==step)
	        {
	            ExecutiveRoutine05ms_B();
	        }
            else if(3==step)
            {
                ExecutiveRoutine05ms_C();
            }
            else if(4==step)
            {
                step = 0;

                ExecutiveRoutine05ms_D();
            }
            else
            {
                if(step>=4) step=0;
            }
	    }

	    ExecutiveRoutine100us();//高精度定时函数(可能存在堵塞)
	}
```



# 控制策略部署

