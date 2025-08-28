# CAN 控制器使用说明

## CAN 控制器功能特性

* 适配ISO 11898-1标准
* 最高达1Mbps比特率
* 多时钟源
* 32个消息对象/邮箱，每个消息对象都有以下特性：
  * 可配置为接收/发送；
  * 可配置为标准11位ID / 29位扩展ID；
  * 支持可编程ID接收掩码；
  * 支持数据帧和遥控帧；
  * 携带0-8字节数据；
  * 奇偶校验配置和数据RAM；
* 每个消息对象具有独立ID掩码；
* 消息对象可编程FIFO模式；
* 自测操作具有可编程回环模式；
* 支持调式的挂起模式；
* 软件模块重启；
* 自动重连——再一个32位定时器判断为掉线后；
* 消息RAM奇偶校验状态机；
* 两个中短线；
* 支持DMA。

## CAN控制器架构

![image-20250822085942952](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250822085942952.png)

### CAN 核心 CAN Core

​	CAN 核心由CAN协议控制器和Rx/Tx移位寄存器组成，用于处理所有ISO 11989 协议功能。

### 消息处理器 Message Handler

​	消息处理器是一个用于控制在Message RAM 和 CAN 核心的Rx/Tx移位寄存器数据交换的状态机。也用于处理接收滤波和当控制寄存器 Control Register 已编程中断请求生。

### 消息内存 Message RAM

​	消息内存可以存储32个CAN消息。

### 寄存器与消息对象访问 Register and Message Object Access

​	通过间接访问消息对象确保消息的一致性。在正常操作下，所有 CPU 和 DMA 都通过接口寄存器 Interface Registers 完成对消息 RAM 的访问。

​	三个接口寄存器组控制 CPU 对消息 RAM 的读和写访问。其中两个接口寄存器组 IF1 和 IF2 用于读和写访问，1个接口寄存器组 IF3 用于只读访问。接口寄存器拥有与消息RAM同样的字长。

​	在指明的测试模式中，消息RAM的内存映射可以被直接访问。

​	![image-20250825105350543](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250825105350543.png)

## 功能说明

​	CAN模块根据ISO 11898-1 协议执行CAN 协议通信。比特率可以编程为最高达1Mbps。一个CAN收发器芯片需要连接到物理层，即CAN总线上。

​	在 CAN 网络上进行通信时，可以对各个消息对象进行配置。消息对象和标识符掩码存储在消息RAM中。

​	所有关于消息处理的功能都被实现在消息处理器中。这些功能包括：接收滤波，在消息RAM和CAN核心之间进行消息的传输；和处理由终端和DMA请求所生产的传输请求。

​	CAN的寄存器组可以由CPU通过模块接口直接访问。这些寄存器被用来控制和配置CAN核心和消息处理器，并且访问消息RAM。

### 配置设备引脚

​	GPIO mux寄存器必须被配置此外设连接到设备引脚。为了避免引脚上出现的小毛病，GPyGMUX 位必须首先被配置（同时保持相关的GPyMUX位为默认0），然后将GPyMUX寄存器写入所需的值。

​	一些I/O的功能由独立于外设的GPIO寄存器设定定义的。对于输入信号，GPIO输入量应该通过设定相应的GPxQSELn寄存器的为11b设置为异步模式。内部输入上拉可以在GPyPUD寄存器进行配置。

​	查看GPIO章节获得更多关于GPIO mux的细节和设定。

### 地址/数据总线桥	

​	CAN模块使用特殊的寻址表来支持字节寻址。推荐只使用32位的

## 操作模式

### 初始化

​	初始化模式由软件（通过设置CAN_CTL寄存器的Init位），硬件复位或者掉线进入。在Init位置1的同时，从CAN总线的接收/发送传输就会停止，并且CAN_TX的状态输出是隐性。CAN错误计数器不会更新。设置Init位不会改变其他任何配置寄存器。

​	为了初始化CAN控制器，CPU必须配置CAN的位时序和用于CAN通信的消息对象。不需要的消息对象，可以通过清除它们的MsgVal位失活。

​	当在CAN控制寄存器的Init和CCE位都置1时，可以访问位时序寄存器配置位时序。

​	清除Init位完成软件的初始化。之后，比特流处理器（BSP）通过等待11个连续隐性比特序列（总线空闲）的出现，将自己同步到CAN总线上的数据传输，然后才能参与总线活动并开始消息传输。

​	消息对象的初始化独立于Init位，然而，所有的消息对象在消息发送开始之前都要配置为特有的标识符或者设置为无效。

​	CPU可以在正常操作模式期间对消息对象进行配置变更。在建立之后，消息对象从接口寄存器到消息RAM进行传输。当正在处理的消息对象号小于等于上一次接收的消息对象时，接收滤波器就会被启用。这保证了即使正改动消息对象仍能保持消息一致性；例如：在有一个挂起的CAN帧接受期间。

### CAN 消息传输 （默认模式）

​	一旦CAN被初始化，并且Init位被复位为0。CAN核心就会自行与CAN总线进行同步然后准备通讯就绪。

​	如果消息通过了接收滤波，那么接收的消息被存在它们想对应的消息对象中。全部消息都被存储在消息对象中，包括：消息ID，数据控制位DLC和最高达8字节的数据。因此，例如，如果使用ID掩码，当一个接收的消息被存储的时候，消息ID位在消息对象中会由掩码变更为“无需关心”。

​	CPU可以在任何时间使用接口寄存器读/写每个消息，因为消息处理器提供了并发访问情况下的消息一致性。

​	将要发送的消息可以被CPU更新。如果对于消息来说存在一个永恒的消息对象（消息ID MSGID，控制位在配置期间设置，并且设置给多个CAN发送器），那么可以只更新数据字节。如果多个发送消息应该被指定到同一个消息对象，那么整个消息对象在被请求的消息传输之前配置完成。

​	多个消息对象的传输可以在同一时间发起。消息对象根据它们内部的优先级按顺序传输。即使一个传输请求扔然挂起，在任何时间消息都可以更新或者设置为无效。然而，如果一个消息在一个挂起的传输已经开始之前变更，那么数据字节会被丢弃。

​	依赖于消息对象的配置，通过接收一个与之匹配的ID的远程帧会自动地请求数据传输。

#### 失能自动重传

​	根据CAN协议，CAN提供一个状态机在传输期间去自动的重发已经失去仲裁或者已经被错误干扰的帧。在传输成功完成之前，不会向用户确认帧传输服务。

​	默认情况下自动重发是使能状态。可以通过设定CAN控制寄存器的DAR位为1去失能。

#### 自动重连

​	在CAN已经进入掉线状态之后，通过重置Init位CPU可以开始一个掉线恢复序列。如果这个过程没实现，模块会保持在掉线状态。

​	CAN提供一个自动的自动在线功能，该功能由ABO位使能。CAN会自动地开始掉线恢复序列。该序列可以被用户定义的时钟周期延迟。

> 注：如果CAN模块由于多个CAN总线错误进入掉线状态，CAN会停止所以总线活动并且自动地置位Init位。一旦Init位由应用清除（或者由于自动重连功能），在回复正常模式之前设备会等待出现129个总线空闲信号（=129个11位连续隐形位）。不可以通过置位或复位Init位缩短掉线恢复序列。在掉线恢复序列的最后，错误计数器会被重置。在Init为重置之后，每一次监测到总线空闲信号，一个Bit0错误码就会被写到错误和状态寄存器。这使得CPU可以检查CAN总线是否被线性堵塞或者连续干扰，并且监控掉线恢复序列的进行。

### 测试模式

​	CAN模块提供几个测试模式，主要用于自测目的。对于所有的测试模式，在CAN控制寄存器中的Test位都要置位为1去使能对CAN测试寄存器的写访问。

#### 静默模式

​	静默模式可以在不发送显性位影响CAN总线的情况下分析通信状况（例如，确认位，过载标志，和激活错误标志）。CAN仍然可以接受有效数据帧和远程帧，但不会发送任何显性位。然而，接收的帧会被内部路由到CAN核心。

![image-20250824190052784](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250824190052784.png)



#### 回环模式

​	回环模式主要用来硬件自测。在该模式，CAN核心使用从Tx输出到Rx输入的内部反馈。发送的消息同时视为接收的消息，如果它们都通过接收过滤就会被存储到消息对象总。CAN_Rx输入引脚的真实值被CAN核心忽视。发放的消息仍然可以被CAN_Tx引脚监控。

​	为了独立于外部仿真，在回环模式CAN核心无视确认错误（在确认槽的数据或者远程帧采样的隐性位）。

![image-20250824191505952](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250824191505952.png)

#### 外部回环模式

​	外部回环模式与回环模式很像。然而，其包括了从CAN核心到Tx引脚信号路径，Tx引脚本身和从Tx引脚到CAN核心的信号路径。当选择外部回环模式，CAN核心被连接到Tx引脚的输入缓冲区。通过该配置，Tx引脚的IO电路可以被测试。外部回环模式可以通过设定测试寄存器的ExL位为1激活。

> 注:当回环模式激活时（LBack 位为1），ExL位会被忽略。

![image-20250824192249090](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250824192249090.png)

#### 静默回环模式

​	同时设置LBack 和 Silent 位就可以组合回环模式和静默模式。在不影响CAN网络的情况下测试CAN的硬件。在该模式，CAN_Rx引脚从CAN核心断连，并且CAN_Tx引脚不会发送显性位。

![image-20250824192940678](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250824192940678.png)

## 多个时钟源

​	CAN位时序时钟通常由系统时钟SYSCLK产生。如果需要，位时序时钟可以直接由外部时钟源（XTAL）产生。

> CAN核心必须编程为每个位至少八个时钟周期。为了实现1Mbps的传输速率需要使用一个8Mhz或者更高频率的晶振。

## 中断特性

​	中断可以由两个中断线产生：CAN0INT和CAN1INT。中断线可以在CAN控制寄存器中设置对应的IE0，IE1为1进行使能。

​	CAN提供了三组中断源：消息对象中断，状态变更中断和错误中断。中断源可以通过中断寄存器的中断ID Int0ID和Int1ID位进行确定。当没有中断挂起时，寄存器保持0。每一个中断线会在中断寄存器中的检测域再次为0（这意味着中断复位）或者IE0和IE1复位之前（中断失能），都会保持激活。在Int0ID域的值0x8000表明因为CAN核心已更新（非必要变更）错误和状态寄存器（错误中断或者状态中断）而挂起的一个中断。该中断有最高优先级。CPU可以通过读错误和状态寄存器更新（复位）RxOK，TxOK和LEC状态位，但CPU的写访问不会产生或者复位一个中断。

​	介于1和最后一个消息对象的序号之间的数值表明了中断源是其中一个消息对象。INT0ID和INT1ID指向有最高优先级的挂起的消息中断。消息对象1有最高优先级，最后一个消息对象有最低优先级。

​	一个从中断源读取消息中断服务流程，也可以在读消息的同时清除消息对象的IntPnd中断挂起标志位。（ClrIntPnd清除中断挂起标志位在IF1或者IF2命令寄存器中）。当中断挂起标志位被清除，中断寄存器指向下一个挂起中断的消息对象。

​	CAN模块有模块级别的中断和确认状态机。要启用CAN0和CAN1中断，必须在CAN_GLB_INT_EN寄存器中设置相应的位。当处理一个中断，使用CAN_GLB_INT_EN 和PIEACK 位确认该中断之前，必须清除独立的消息或者状态变更标志位。

### 消息对象中断

​	消息对象中断由消息对象的事件产生。由IntPnd，TxIE和RxIE 控制。消息对象中断可以路由到CAN0INT或CAN1INT线，这是由中断多路复用器寄存器控制的。

> 注意：写CAN_IFnMCTL的IntPnd位会强制产生中断。

### 状态变更中断

​	在错误和状态寄存器中的RxOK，TxOK和LEC事件都属于状态变更中断。状态变更中断组可以通过设置CAN控制寄存器的SIE位使能。如果SIE置位，每一个CAN帧，独立的总线错误，有效的CAN通信和独立的消息RAM配置都会产生一个状态变更中断。状态变更中断只能被路由到中断线CAN0INT(必须通过在CAN_CLT寄存器的IE0位置1使能)。

### 错误中断

​	PER，BOff 和 EWarn事件都属于错误中断。错误中断组可以通过置位EIE位使能。同样，错误中断也只能路由到CAN0INT中断线。

### DCAN中断的PIE术语

| Interrupt | CANA   | CANB   |
| --------- | ------ | ------ |
| CANINT0   | CANA_0 | CANB_0 |
| CANINT1   | CANA_1 | CANB_1 |

### 中断拓扑

​	邮箱的中断可以路由到CAN0INT或者CAN1INT线，而错误中断和状态变更中断只能路由到CAN0INT线。

![image-20250824203022627](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250824203022627.png)

![image-20250824203034023](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250824203034023.png)

## DMA特性



## 奇偶校验状态机



## 调试模式

​	模块支持通过提供像中断CAN活动和使得调试器访问消息RAM内容的功能的外部调试单元的使用。当一个外部的调试器已连接并且内核暂停运行时自动进入调试模式。

​	在调试模式开始之前，电路会一直等待知道一次传输的开始，接收的完成或者识别到总线空闲状态。如果IDS位置位，调试器会立即中断当前的传输和接收。之后，由在CAN控制寄存器的InitDbg标志位指示CAN进入调试模式。在调试期间，所有的CAN寄存器都可以被访问。读取保留的位会返回0，写保留的位无效。同时，消息RAM被内存映射，所以允许外部调试器读取消息RAM。

![image-20250826210441775](C:\Users\JetzeyChen\AppData\Roaming\Typora\typora-user-images\image-20250826210441775.png)

> 注意：在调试期间，消息RAM不可被IFx寄存器组访问。在调试模式写控制寄存器可以影响CAN状态机，甚至消息处理。

为了支持调试，如下CAN寄存器的自动清除功能被失能：

* 错误状态寄存器（由读访问清除状态标志）
* IF1/IF2命令寄存器（由读或者写访问清除DMA激活标志位）

## 模块初始化

​	在硬件复位之后，CAN控制寄存器的Init位被置位并且所有的CAN协议功能都会失能。在CAN协议功能使能之前可以完成位时和消息对象的配置。

​	通过硬件复位，message对象的MsgVal、NewDat、IntPnd和TxRqst位被重置为0。**消息对象的配置是通过编程IF1/IF2接口寄存器组之一的掩码、仲裁、控制和数据位为所需的值来完成设置**。通过将消息对象编号写入对应的IF1/IF2命令寄存器的比特位[7:0]，IF1/IF2接口寄存器的内容被加载到消息内存中对应地址的消息对象中。

​	位时序的配置要求CAN控制寄存器中的Init位置1另外设置CCE也置1。这对于消息对象的配置来说不是必需的。

​	当CAN控制寄存器中的Init位被清除后，CAN核的CAN协议控制器状态机和消息处理程序状态机开始控制CAN内部的数据流。接收到的消息，通过接受过滤的消息存储到消息RAM中；带有待定传输请求的消息被加载到CAN核心的移位寄存器中，并使用CAN总线进行传输。

​	当CPU清除Init和CCE时，可以同时使能中断线（将IE0和IE1设置为1）。状态中断“EIE”和“SIE”可以同时启用。

​	CAN通信可以控制中断驱动或轮询模式。中断寄存器指向那些IntPnd = 1的message对象。即使到CPU的中断线被禁用（IE0和IE1为0），寄存器也会被更新。

​	CPU可以从NewData寄存器和传输请求寄存器中并行轮询所有MessageObject的NewDat和TxRqst位。如果所有传输对象都按较小的编号分组所有接收对象都按较高的数字分组，则轮询更容易。

## 配置消息对象

​	整个消息RAM都应该在初始化结束之前配置。在CAN通信期间也可以改变消息对象的配置。

#### 数据帧发送对象的配置

| MsgVal |  Arb  | Data  | Mask  | EoB  | Dir  | NewDat | MsgLst | RxIE | TxIE  | IntPnd | RmtEn | TxRqst |
| :----: | :---: | :---: | :---: | :--: | :--: | :----: | :----: | :--: | :---: | :----: | :---: | :----: |
|   1    | appl. | appl. | appl. |  1   |  1   |   0    |   0    |  0   | appl. |   0    | appl. |   0    |

* 仲裁位arbitration bits(ID[28:0]和Xtd位)由应用提供。定义了ID和外发消息的类型。如果使用11位ID（Xtd = 0），编程到ID[28:18]。在该情况下，ID[17:0]被忽略。
* 数据寄存器（DLC[3:0]和数据0-7）由应用给出。**TxRqst 和 RmtEn 应该在数据有效之前不能置位**。
* 如果TxIE置位，在一次消息对象的成功传输之后IntPnd位被置位。
* **如果RmtEn置位，接收一个匹配的远程帧会导致TxRqst位置位；远程帧会自主地由一个数据帧应答**。
* 掩码位(Msk[28:0],UMask,MXtd和Mdir位)可以允许一组有相似一ID远程帧置位TxRqst位。Dir位不应被遮掩。如果没有远程帧被允许置位TxRqst位（RmtEn = ‘0’），ID遮掩必须失能（UMask = ‘0’）。

#### 远程帧发送对象的配置

​	没有必要去为远程帧的传输配置传输对象。为接收消息对象设置TxRqst将导致传输一个与配置此接收对象的数据帧具有相同标识符的远程帧。

#### 数据帧单个接收对象配置

| MsgVal |  Arb  | Data  | Mask  | EoB  | Dir  | NewDat | MsgLst | RxIE | TxIE  | IntPnd | RmtEn | TxRqst |
| :----: | :---: | :---: | :---: | :--: | :--: | :----: | :----: | :--: | :---: | :----: | :---: | :----: |
|   1    | appl. | appl. | appl. |  1   |  0   |   0    |   0    |  0   | appl. |   0    |   0   |   0    |

* 仲裁位arbitration bits(ID[28:0]和Xtd位)由应用提供。定义了ID和外发消息的类型。如果使用11位ID（Xtd = 0），编程到ID[28:18]。在该情况下，ID[17:0]被忽略。当一个有11位ID的数据帧被接收，ID[17:0]会被置‘0’。
* **当消息处理器在消息对象存储一个数据帧，它会存储接收数据长度码和相对应的数据字节数。如果数据长度码少于8，剩余的消息对象字节会被覆盖为非指定值**。
* 掩码位(Msk[28:0],UMask,MXtd和Mdir位)被用来（UMask = ‘1’）允许一组有相似ID的数据帧被接收。在典型应用中Dir位不应该被遮掩。如果掩码位的一些位被置为“无关紧要”，相关的仲裁寄存器的位会被存储的数据帧的位覆盖。
* 如果RxIE置位，当一个接收的数据帧被接受并且存储在消息对象中时IntPnd位置1。
* **如果TxRqst置位，ID与实际存储的仲裁位一致的远程帧的传输会被触发**。如果掩码位（UMask = ‘1’）被用来做接收过滤，仲裁位的内容可能会被改变。

#### 远程帧的单个接收对象配置

| MsgVal |  Arb  | Data  | Mask  | EoB  | Dir  | NewDat | MsgLst | RxIE | TxIE  | IntPnd | RmtEn | TxRqst |
| :----: | :---: | :---: | :---: | :--: | :--: | :----: | :----: | :--: | :---: | :----: | :---: | :----: |
|   1    | appl. | appl. | appl. |  1   |  1   |   0    |   0    |  0   | appl. |   0    |   0   |   0    |

* 远程帧接收对象可用来监控CAN总线上的远程帧。远程帧存储在接收对象不会触发数据帧的传输。远程帧的接收对象可扩展到FIFO缓冲区。
* **UMask必须设置为‘1‘。掩码位(Msk[28:0],UMask,MXtd和Mdir位)应该设置为“必须匹配”或者“无关紧要”用来允许一组有相似ID的远程帧被接受**。在典型应用中Dir位不应该被遮掩。
* 仲裁位arbitration bits(ID[28:0]和Xtd位)由应用提供。定义了ID和被接收的远程帧的类型。如果Mask位的一些位被设置为’无关紧要‘，仲裁位相应的位会被存储的远程帧的位覆盖。如果使用11位ID（Xtd = 0），编程到ID[28:18]。在该情况下，ID[17:0]被忽略。当一个有11位ID的数据帧被接收，ID[17:0]会被置‘0’。
* 数据长度码（DLC[3:0]）由应用给出。当消息处理器在消息对象中存储一个远程帧，会存储接收的数据长度码。数据对象的数据字节会保持不变。
* 如果RxIE被置位，当一个接收的远程帧被接受IntPnd位会被置位并且存储在消息对象中。

#### FIFO缓冲区的配置



## 消息处理

​	当初始化完成，CAN模块会同步自身到CAN总线的通信。执行接收消息的接受过滤和存储被接收的帧到指定的消息对象。**应用必须更新要将要发送的消息的数据和使能或者请求它们的传输**。**当一个匹配的远程帧接收时自动产生一个传输请求**。

​	应用可以读取接收的消息。对于同一个消息对象如果在下一个消息接收之前没有读取消息，那么消息就会被覆盖。**消息可以由中断驱动或者轮询NewDat位之后读取**。

#### 消息处理器总览

​	消息处理器状态机控制CAN核心的接收/发送移位寄存器与消息RAM之间的数据传输。消息处理状态机执行以下几个任务：

* 从消息RAM到CAN核心的数据传输（发送消息）；
* 从CAN核心到消息RAM的数据传输（接收消息）；
* 从CAN核心到接收过滤单元的数据传输；
* 扫描消息RAM获取一个匹配的消息对象（接收过滤）；
* 在上一次扫描发现有一致或更高优先级的消息对象时，被IF1/IF2更改之后，扫描消息对象；
* 处理TxRqst标志；
* 处理中断标志。

消息处理器寄存器将消息对象的状态标志分为如下几个主题存储：

* 发送请求标志
* 新数据标志
* 中断挂起标志
* 消息有效寄存器

​	**取代通过分别使用相应的IFx寄存器收集上面所列出的每一个消息对象的状态信息，消息处理器寄存器提供更快更简单的方式去获得总览**。

​	所有消息处理器寄存器都是只读的。

#### 发送/接收优先级

​	**消息对象的接收/发送优先级与消息序号绑定，而不是CAN ID**。消息对象1有最高的优先级，与此同时消息对象32有最低的优先级。如果超过一个发送请求挂起，将会根据相应的消息对象的优先级提供服务，例如，最高优先级的消息可以放入最低优先级的消息对象。

​	**数据帧和远程帧帧的接收过滤也是按上升排序的消息对象完成，所以一个已经被消息对象接受的帧不能被更高消息序号的消息对象接受**。对于需要记录完整的CAN总线通信，最后一个消息对象可以配置用于接受任何不被其他数据对消接受的数据帧和远程帧。

#### 事件驱动消息发送的CAN通信

​	如果CAN核心的一位寄存器准备加载并且接口寄存器和消息RAM之间没有数据传输，消息有效寄存器的MsgVal位和发送请求寄存器的TxRqst位被评估。挂起传输请求的最高优先级的有效消息对象被消息处理器装载到一位寄存器中并且开始发送。消息对象的NewDat被复位。

​	如果传输完成并且自从传输开始之后没有新消息写入到消息对象，TxRqst位会被复位。如果TxIE置1，成功传输之后会置位IntPnd。如果CAN丢失仲裁或者如果传输过程中产生错误，在CAN总线可用情况下尽快重传消息。如果与此同时有更高优先级的消息传输已经被请求，根据优先级进行消息传输。

​	如果通过置位CAN控制寄存器的DAR位失能自动重传模式，消息控制寄存器和接口寄存器组的TxRqst和NewDat位的行为将会如下：

* 当传输开始，在NewDat保持为1的同时，相应的接口寄存器组的TxRqst位复位。
* 当传输已经成功完成，NewDat位复位。

​	当一次传输失败，NewDat保持置位。为了开始重新开始传输，应用必须去再次置位TxRqst。

​	**接收远程帧并不需要接收对象。如果匹配的传输对象的RmtEn位置位，它们会自动触发一个数据帧的传输**。

#### 更新发送对象

​	CPU可以使用IF1和IF2接口寄存器组再任何时间发送镀锡的数据字节；再更新之前，MsgVal 和 TxRqst两者都不需要被复位。

​	即使只有一部分数据字节需要更新，对应的IF1/IF2的数据A寄存器或者数据B寄存器中的四个字节再发送到消息对象之前都应该有效。再CPU写新数据字节之前，要么CPU必须通过IF1/IF2数据寄存器写全部的四个字节，要么消息对象被传输的到IF1/IF2数据寄存器。

​	当只有数据字节被更新时，IF命令寄存器的[23:16]位域先被写位0x87，然后IF命令寄存器的[7:0]的位域被消息对象的序号填充，更新数据字节和用NewdDat设定TxRqst，是并行的。

​	**为了避免在已经进行的传输结尾TxRqst复位同时数据被更新，在事件驱动CAN通信中NewDat必须和TxRqst同时置位。**

#### 变更发送对象

​	如果使用的消息对象数量不满足被用来当作永久消息对象，发送对象可以被动态管理。CPU可以写整个消息（包括ID，DLC，DATA）到接口寄存器。**在传输整个消息对象的内容到消息对象之前，IF命令寄存器的[23:16]位域应被设置位0xB7**。在这个操作之前MsgVal 和 TsRqst都不需要复位。

​	如果先前请求传输的消息对象已在进行但没有完成，传输将会继续；然而，如果传输被干扰了不会重发。

​	**只更新被发送消息的数据字节，需要设置IF命令寄存器[23:16]位域位0x87**。

>注：在更新发送对象之后，接口寄存器组保留实际对象内容的副本，包括没有被更新的部分。

#### 接收消息的接受过滤

​	当一个输入消息的仲裁和控制位（ID+IDE+RTR+DLC）完全移位到CAN核心的移位寄存器时，消息处理器开始去扫描消息RAM寻找一个匹配的有效消息对象：

* 接收滤波器加载来自CAN核心移位寄存器的仲裁位（ID+IDE+RTR）。
* 然后消息对象1的仲裁位和掩码位被装在到接收滤波器中，并且被与来自移位寄存器的相关位进行比较。这将会重复进行直到一个匹配的消息对象被找到，或者到达了消息RAM的结尾。
* 如果产生匹配，扫描就会停止并且消息处理器根据接收帧的类型进行处理。

#### 数据帧的接收

​	消息处理器存储来自CAN核心移位寄存器的消息到对应的消息RAM中的消息对象。不只是数据字节，全部的仲裁位和数据长度码都被存储到响应的消息对象。这确保了即使使用仲裁掩码寄存器，也会保持数据字节跟ID之间相关联。

​	NewDat位置位表明新的数据已经被接收，CPU没有看过。当CPU读取消息对象时必须复位NewDat。如果在接收的同时NewDat已经置位，MsgLst置位表明先前的数据丢失。如果RxIE位置位，导致中断寄存器指向该消息对象，然后IntPnd位置位。

​	复位消息对象的TxRqst位避免在请求的数据帧刚被接收的同时，发送一个远程帧。

#### 远程帧的接收

​	当远程帧被接受，匹配的消息对象考虑以下三种不同配置：

1. Dir = 1（发送），RmtEn = 1， Umask = 1 or 0

​	接收到匹配的远程帧后置位消息对象的TxRqst。消息对象的其余部分保持不变。

1. Dir = 1，RmtEn = 0， Umask = 0

​	远程帧被忽视，消息对象保持不变。

1. Dir = 1，RmtEn = 0， Umask = 1

   远程帧被当作类似的数据帧。在接收到匹配的远程帧时，消息对象的TxRqst位复位。将仲裁位域和控制位域从移位寄存器存储到消息RAM的消息对象中，并且消息对象的NewDat 置位。消息对象的数据字节保持不变。

#### 读取接收消息

​	CPU可以在任意时间通过IFx寄存器组去读取接收的消息，由消息处理状态机确保数据一致性。

​	通常CPU写IFxCMD命令寄存器的[23:16]为0x7F和[7:0]写为消息对象序号。一并传输整个接收的消息从消息RAM到接口寄存器组。另外，消息RAM中的NewDat 和 IntPnd位被清除（非接口寄存器组）。在IFxMCTLR消息控制寄存器的这写位的值总是反应这些位重置之前的状态。

​	如果消息对象使用掩码作接收过滤，被接受的匹配对象不一致的仲裁位将显示。

​	NewDat的实际值表示是否在上一次读取消息对象之后有消息被接收。MsgLst的实际值表示是否在上一次读取消息对象之后有超过一个消息被接收。MsgLst不会自动复位。

#### 为接收对象请求新数据

​	通过远程帧的方式，CPU可以向其他CAN节点向接收对象请求提供新的数据。设置接收对象的TxRqst位会导致与接收对象ID一致的远程帧的传输。该远程帧触发其他CAN节点开始传输匹配的数据帧。如果在匹配的远程帧发送之前，匹配的数据帧已经接收，TxRqst位自动的复位。

​	在不改变消息对象的内容的情况下设置TxRqst，需要IFxCMD命令寄存器的[23:16]位域值为0x84。

## CAN位时序



## 消息接口寄存器组



## 消息RAM



## 软件



## CAN寄存器







# CAN 驱动开发

## CAN GPIO配置

​	以CAN的Tx与Rx引脚配置为例，分别C2000Ware提供的库函数，GPIO_setPinConfig(pinConfig);

```  c
void CANA_GPIO_Init(void) {
  GPIO_setPinConfig(CANA_RX_PIN);
//   GPIO_setPadConfig(CANA_RX, GPIO_PIN_TYPE_STD);
//   GPIO_setQualificationMode(CANA_RX, GPIO_QUAL_ASYNC);

  GPIO_setPinConfig(CANA_TX_PIN);
//   GPIO_setPadConfig(CANA_TX, GPIO_PIN_TYPE_STD);
//   GPIO_setQualificationMode(CANA_TX, GPIO_QUAL_ASYNC);
}
```

​	**调用该函数则直接将该引脚连接到IO口并且配置为相应的功能**，如果GPIO_30_CANA_RX，则将30Pin设置为CANA_Rx的复用功能，其相较于ST库函数较为方便。

## CAN Module配置

### 初始化CAN

 	初始化CAN模块，C2000Ware提供的库函数中基本上都存在参数校对ASSERT()函数;（这对于开发来说是非常有帮助的，有助于避免一些不太注意到的错误产生）。

```c
void CAN_initModule(uint32_t base)
{
    ASSERT(CAN_isBaseValid(base));//参数核对

    HWREGH(base + CAN_O_CTL) |= ((uint16_t)CAN_CTL_INIT |
                                 (uint16_t)CAN_INIT_PARITY_DISABLE);//失能CAN校验和初始化CAN
    CAN_initRAM(base);//初始化消息RAM

    HWREGH(base + CAN_O_CTL) |=  CAN_CTL_SWR;//软件强制CAN复位
    //
    // Delay for 14 cycles
    //
    SysCtl_delay(1U);

    HWREGH(base + CAN_O_CTL) |= CAN_CTL_CCE;//使能配置变更
}
```

​	HWREGH()是定义在hw_types.h的头文件中的宏函数，其本质是*（volatile uint16_t *(x)）对寄存器地址低16位的访问。CAN控制寄存器CAN_CTL的上述使用相关位解释：

* CAN_CTL_INIT：

  * 1：无视CAN总线上的活动，用来保证CAN位时序和消息RAM的初始化，当掉线时也会由硬件自动置位；
  * 0：正常进行CAN总线通信。

* CAN_CTL_PMD：

  * 0x5：校验失能；
  * 其他值：使能。

* CAN_CTL_SWR:

  * 1：强制执行软件复位，在复位执行的一个时钟周期后自动复位；
  * 0：无操作。

  > 注：软件复位的执行操作是：1.置位Init，2.置位SWR（SWR是由Init写保护的）。使用SWR复位模块，不会丢失用户配置。

* CAN_CTL_CCE：

  * 1：当Init = 1时，CPU可以对配置寄存器进行写访问；
  * 0：不可写访问配置寄存器

### 配置CAN波特率

​	由CAN_serBitRate()函数对CAN通信的波特率进行配置。该函数本质是通过时钟频率以及目标比特率和位时间片数量，推算出位时序配置参数。其会调用CAN_setBitTiming()函数，配置位时序。

```c
void CAN_setBitRate(uint32_t base, uint32_t clockFreq, uint32_t bitRate,uint16_t bitTime)
{
    uint16_t brp;
    uint16_t tPhase;
    uint16_t phaseSeg2;
    uint16_t tSync = 1U;
    uint16_t tProp = 2U;
    uint16_t tSeg1;
    uint16_t tSeg2;
    uint16_t sjw;
    uint16_t prescaler;
    uint16_t prescalerExtension;

    //
    // Check the arguments.
    //
    ASSERT(CAN_isBaseValid(base));
    ASSERT((bitTime > 7U) && (bitTime < 26U));
    ASSERT(bitRate <= 1000000U);

    //
    // Calculate bit timing values
    //
    brp = (uint16_t)(clockFreq / (bitRate * bitTime));
    tPhase = bitTime - (tSync + tProp);
    if((tPhase / 2U) <= 8U)
    {
        phaseSeg2 = tPhase / 2U;
    }
    else
    {
        phaseSeg2 = 8U;
    }
    tSeg1 = ((tPhase - phaseSeg2) + tProp) - 1U;
    tSeg2 = phaseSeg2 - 1U;
    if(phaseSeg2 > 4U)
    {
        sjw = 3U;
    }
    else
    {
        sjw = tSeg2;
    }
    prescalerExtension = ((brp - 1U) / 64U);
    prescaler = ((brp - 1U) % 64U);

    //
    // Set the calculated timing parameters
    //
    CAN_setBitTiming(base, prescaler, prescalerExtension, tSeg1, tSeg2, sjw);
}
```

### 配置消息对象Message Objects

​	配置消息对象是通过调用CAN_setupMessageObject()来实现的，其中包括8个参数：

* CAN模块：CANA/CANB；
* 消息对象序号：1~32，数字越小优先级越高；
* 消息ID：消息的仲裁位；
* 消息帧类型：数据帧/远程帧；
* 消息对象类型：接收对象/发送对象/接收远程帧对象/发送远程帧对象；
* 消息ID掩码：用于使多个ID匹配同个消息对象；
* 消息中断标志：发送中断/接收中断；
* 消息长度：对发送对象有效。

``` c
void CAN_setupMessageObject(uint32_t base, uint32_t objID, uint32_t msgID,
                       CAN_MsgFrameType frame, CAN_MsgObjType msgType,
                       uint32_t msgIDMask, uint32_t flags, uint16_t msgLen)
{
    uint32_t cmdMaskReg = 0U;
    uint32_t maskReg = 0U;
    uint32_t arbReg = 0U;
    uint32_t msgCtrl = 0U;

    ASSERT(CAN_isBaseValid(base));
    ASSERT((objID <= 32U) && (objID > 0U));
    ASSERT(msgLen <= 8U);

    //等待命令寄存器完成上一次指令操作，进入空闲状态
    while((HWREGH(base + CAN_O_IF1CMD) & CAN_IF1CMD_BUSY) == CAN_IF1CMD_BUSY)
    {
    }

    switch(msgType)
    {
        case CAN_MSG_OBJ_TYPE_TX://消息接收对象
        {
            arbReg = CAN_IF1ARB_DIR;//设置ID中的Dir为1表示发送
            break;
        }

        case CAN_MSG_OBJ_TYPE_RXTX_REMOTE://消息接收发送远程帧对象
        {
            arbReg = CAN_IF1ARB_DIR;
            msgCtrl = (uint32_t)((uint32_t)CAN_IF1MCTL_RMTEN |
                                 (uint32_t)CAN_IF1MCTL_UMASK);//启用自动发送匹配远程帧，使用ID掩码
            break;
        }

        //
        // Transmit remote request message object (CAN_MSG_OBJ_TYPE_TX_REMOTE)
        // or Receive message object (CAN_MSG_OBJ_TYPE_RX).
        //
        default:
        {
           arbReg = 0U;
           break;
        }
    }

    if(frame == CAN_MSG_FRAME_EXT)//扩展帧
    {
        if((flags & CAN_MSG_OBJ_USE_ID_FILTER) == CAN_MSG_OBJ_USE_ID_FILTER)
        {
            maskReg = msgIDMask & CAN_IF1MSK_MSK_M;//使用ID掩码
        }
        arbReg |= (msgID & CAN_IF1ARB_ID_M) | CAN_IF1ARB_MSGVAL |
                  CAN_IF1ARB_XTD;//设置ID，消息对象有效，且为扩展帧
    }
    else //标准帧
    {
        if((flags & CAN_MSG_OBJ_USE_ID_FILTER) == CAN_MSG_OBJ_USE_ID_FILTER)
        {
           maskReg = ((msgIDMask << CAN_IF1ARB_STD_ID_S) &
                      CAN_IF1ARB_STD_ID_M);
        }
        arbReg |= ((msgID << CAN_IF1ARB_STD_ID_S) & CAN_IF1ARB_STD_ID_M) |
                  CAN_IF1ARB_MSGVAL;
    }

    maskReg |= (flags & CAN_MSG_OBJ_USE_EXT_FILTER);//使用扩展ID掩码

    maskReg |= (flags & CAN_MSG_OBJ_USE_DIR_FILTER);//使用方向掩码

	//其他过滤器设置
    if(((flags & CAN_MSG_OBJ_USE_ID_FILTER) |
        (flags & CAN_MSG_OBJ_USE_DIR_FILTER) |
        (flags & CAN_MSG_OBJ_USE_EXT_FILTER)) != 0U)
    {
        msgCtrl |= CAN_IF1MCTL_UMASK;
    }

	//配置消息长度，只对发送对象有效
    if((msgType == CAN_MSG_OBJ_TYPE_TX) ||
        (msgType == CAN_MSG_OBJ_TYPE_RXTX_REMOTE))
    {
        msgCtrl |= ((uint32_t)msgLen & CAN_IF1MCTL_DLC_M);
    }

    //
    // If this is a single transfer or the last mailbox of a FIFO, set EOB bit.
    // If this is not the last entry in a FIFO, leave the EOB bit as 0.
    //
    if((flags & CAN_MSG_OBJ_FIFO) == 0U)
    {
        msgCtrl |= CAN_IF1MCTL_EOB;
    }

    msgCtrl |= (flags & CAN_MSG_OBJ_TX_INT_ENABLE);//发送中断使能

    msgCtrl |= (flags & CAN_MSG_OBJ_RX_INT_ENABLE);//接收中断使能

    cmdMaskReg |= CAN_IF1CMD_ARB;//访问消息RAM中消息对象的ARB位域
    cmdMaskReg |= CAN_IF1CMD_CONTROL;//访问消息对象的control位域
    cmdMaskReg |= CAN_IF1CMD_MASK;//访问消息对象的掩码位域
    cmdMaskReg |= CAN_IF1CMD_DIR;//设置访问为写访问，IFx接口寄存器组内容会被写入消息对象

    HWREG_BP(base + CAN_O_IF1MSK) = maskReg;//配置掩码数据到IF1MSK寄存器
    HWREG_BP(base + CAN_O_IF1ARB) = arbReg;//配置仲裁ID数据到IF1ARB寄存器
    HWREG_BP(base + CAN_O_IF1MCTL) = msgCtrl;//配置消息控制信息到IF1MCTL寄存器

    //
    // Transfer data to message object RAM
    //
    HWREG_BP(base + CAN_O_IF1CMD) =
    cmdMaskReg | (objID & CAN_IF1CMD_MSG_NUM_M);//配置命令，将消息对象为objID的对象更新上述内容
}
```

## CAN中断配置

​	CAN控制器包括的中断有：消息对象中断、错误和状态变更中断、CAN外设全局中断。

### 消息对象中断

​	消息对象的中断则为接收中断与发送中断，该配置在消息对象配置时完成，相应中断置位后，如果完成相关操作便会置位IntPnd。每个消息对象的IntPnd位都会路由到相应的中断寄存器CAN_INT中。

### 错误和状态变更中断

​	该中断属于CAN控制内部中断，如果相应的中断使能，那么可以在错误和状态变更寄存器CAN_ES中查看中断是何种中断。CAN内部中断由函数CAN_enableInterrupt()函数使能，其内部是对CAN控制寄存CAN_CTL中的相关位进行配置。

```c
static inline void CAN_enableInterrupt(uint32_t base, uint32_t intFlags)
{
    ASSERT(CAN_isBaseValid(base));
    ASSERT((intFlags & ~(CAN_INT_ERROR | CAN_INT_STATUS | CAN_INT_IE0 |
                         CAN_INT_IE1)) == 0U);

    HWREG_BP(base + CAN_O_CTL) |= intFlags;
}
```

### CAN全局中断

​	CAN外设中断也可以视作CPU的CAN全局中断。其中断线有两条分别是CANINT0、和CANINT1。其通过调用CAN_enableGlobalInterrupt()函数实现中断使能。本质是对CAN全局中断使能寄存器CAN_GLB_INT_EN的配置。通过此配置后CAN控制的消息对象中断、错误中断和状态变更中断才会路由到两个中断线。

```c
static inline void CAN_enableGlobalInterrupt(uint32_t base, uint16_t intFlags)
{
    ASSERT(CAN_isBaseValid(base));
    ASSERT((intFlags & ~(CAN_GLOBAL_INT_CANINT0 |
                         CAN_GLOBAL_INT_CANINT1)) == 0U);

    HWREGH(base + CAN_O_GLB_INT_EN) |= intFlags;
}
```

​	对于全局中断的配置还需要使能到CPU，这一个过程就需要引入PIE外设中断扩展的电路功能。其相当于ST控制器内部的NVIC，负责外设中断到CPU之间的路由。其初始化以及使能过程如下：

```c
void CAN_Init(void) {
  // Initialize PIE and clear PIE registers. Disables CPU interrupts.
  Interrupt_initModule();

  // Initialize the PIE vector table with pointers to the shell Interrupt
  // Service Routines (ISR).
  Interrupt_initVectorTable();

  // Enable Global Interrupt (INTM) and realtime interrupt (DBGM)
  EINT;
  ERTM;
}
```

​	首先初始化PIE，然后初始化PIE向量表，其内部包含相关外设的ISR入口地址。然后使能全局中断，使能调试中断。

CAN相关的全部中断都可以在CAN_INT中断寄存器查看。

![image-20250828172035870](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250828172035870.png)

* INT1ID位域：只能用于指示被配置到CANINT1中断线上的消息对象中断，当该位域为0x00时，表明INT1无中断；当为0x1-0x20时，则分别对应挂起中断的消息对象序号，其他值无效。

> 注：将消息对象分配到CANINT1线上需要配置CAN_IP_MUX寄存器中相应的位，每一位对应每一个消息对，该位置1则表示中断路由到CANINT1线上，为0则路由到CANINT0线。

* INT0ID位域：用于表示CANINT0线上的消息对象中断，CAN错误中断和状态变更中断都只能路由到CANINT0线。当该位域为0x8000时，表示CAN_ES中的值发送改变，可以获取CAN_ES的值判断是何种中断。上述中断具有最高优先级。其他功能与INT1ID位域一致。

## CAN 中断服务函数ISR配置 

​	首先需要声明中断服务函数的函数原型，其中函数名称可以任取，__interrup关键字必须有。

```c
__interrupt void canaISR(void);//CANA中断服务函数
```

​	然后注册声明的函数原型到PIE的向量表中，由函数Interrupt_register()函数完成。

```c
static inline void Interrupt_register(uint32_t interruptNumber, void (*handler)(void))
{
    uint32_t address;
	//根据中断号计算其在向量表中的位置
    address = (uint32_t)PIEVECTTABLE_BASE +
              (((interruptNumber & 0xFFFF0000U) >> 16U) * 2U);
    
    EALLOW;//宏函数，使能对保护寄存器的写访问
    HWREG(address) = (uint32_t)handler;//将注册的中断函数地址写入向量表中
    EDIS;//宏函数，失能对保护寄存器的写访问
}
```

​	之后就是使能该中断号。

```c
void Interrupt_enable(uint32_t interruptNumber)
{
    bool intsDisabled;
    uint16_t intGroup;
    uint16_t groupMask;
    uint16_t vectID;

    vectID = (uint16_t)(interruptNumber >> 16U);//计算向量号ID
	
    //使能全局中断并保留状态
    intsDisabled = Interrupt_disableGlobal();

    //
    // PIE Interrupts
    //
    if(vectID >= 0x20U)
    {
        intGroup = (uint16_t)(((interruptNumber & 0xFF00UL) >> 8U) - 1U);//计算向量组
        groupMask = (uint16_t)1U << intGroup;

        HWREGH((PIECTRL_BASE + PIE_O_IER1 + (intGroup * 2U))) |=
            (uint16_t)1U << ((interruptNumber & 0xFFU) - 1U);

        //
        // Enable PIE Group Interrupt
        //
        IER |= groupMask;//失能PIE组中断
    }

    //
    // INT13, INT14, DLOGINT, & RTOSINT
    //
    else if((vectID >= 0x0DU) && (vectID <= 0x10U))
    {
        IER |= (uint16_t)1U << (vectID - 1U);
    }
    else
    {

    }
	
    //使能被失能的中断
    if(!intsDisabled)
    {
        (void)Interrupt_enableGlobal();
    }
}
```

​	最后便是编写中断服务函数ISR的具体实现流程。

>注：对于中断，C2000不支持中断嵌套；所以每个中断都必须只进行必要的状态变更操作，严禁出现延迟等长时间等待操作。在中断与中断之外同时访问的变量，要注意边界保护，避免产生访问冲突。



### 配置掩码Mask 

​	掩码配置存储在掩码寄存器IFx_MSK中。如图

![image-20250827152223625](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250827152223625.png)

* MXtd位表示扩展位是否用于接收滤波。对于扩展帧掩码而言，Mxtd = ‘1’。
* Msk位为1的bit表示，该位被用来做接收滤波，不匹配的ID不会被该消息对象接收。
* MDir位表示Dir是否用于接收滤波，通常来说都该位为‘0’。

​	对与一个ID为0x1801 D0EF的扩展帧，其32位表示如下。

![image-20250827152622710](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250827152622710.png)

​	对于发送对象而言，如果ID为0x1801D0EF，0x1803D0EF，0x1805D0EF……的发送消息，其可以通过给发送对象配置一个掩码9FF0 FFFF，而使得上述ID的消息可以共用相同的发送对象，减少消息对象的使用。

![image-20250827153138127](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20250827153138127.png)







## 



## CAN 应用层（测试用）









# CAN通信调试

 ## 中断驱动通信

 	当开启状态与错误中断使能时，如果错误与状态（CAN_ES寄存器复位后为0x7）发生变更了(不为0x7)，最外部的中断寄存器CAN_INT会指向（此时值为0x8000）优先级最高的错误与状态变更寄存器CAN_ES。则由消息对象挂起的中断会被屏蔽掉。

