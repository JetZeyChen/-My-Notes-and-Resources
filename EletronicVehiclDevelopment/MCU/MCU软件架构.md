

# MCU需求解析

 	MCU就是根据VCU发送的控制命令，其中包括电机使能，主动放电，控制模式，目标转速/目标转矩等数据，对VCU的命令进行响应，同时需要确保电机在正常工况下完成需求，尽可能让驾驶更加平滑舒适。

​	对于驾驶平滑度与舒适性方面的内容，就需要涉及到电机本身参数，与实际运行环境的影响。

![img](https://picx.zhimg.com/v2-6779b0f15b8c77d23281b9c946dd2791_1440w.jpg)

# MCU软件架构

## 主框架

### 主循环subMain.c

​	![image-20250918133108545](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250918133108545.png)

* mainFuncEach05ms 周期0.5ms

> ```c
> void mainFuncEach05ms(void){
>    	EepromDeal();                   // EEPROM处理，即参数的读写
>     OscScopeLogicDeal();
> #if 1   //+= 调试，看eepromChkWord是否被无意改变
>     if ((para.elem.eepromCheckWord1 != EEPROM_CHECK1)
>         && (para.elem.eepromCheckWord2 != EEPROM_CHECK2)
>         ){
>         errorCode = 100;
>     }
> #endif
> }
> ```

* mainFunc05mcB  周期2ms

> ```c
> ```
>
> 

* mainFunc05mcC 周期2ms
* mainFunc05mcD 周期2ms
