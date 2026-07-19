# MCU内核架构
##Cortex内核分类及特征
- Cortex-A（Application）:高时钟频率，长流水线，高性能；应用于移动计算、智能手机
- Cortex-R(Real-time):较高时钟频率，较长的流水线，实时性强；应用于军工、汽车电子
- Cortex-M(Microcontroller):时钟频率较低，通常较短的流水线，超低耗频率；应用于工控、传感器、消费电子
## Cortex-M3/4/7介绍
<img width="1206" height="534" alt="image" src="https://github.com/user-attachments/assets/d1ae5195-cb16-4c26-89f0-4fb1ca8db164" />


## STM32初识
### 什么是STM32
ST：意法半导体  M:MCU/MPU  32：32位
### STM芯片分类
<img width="1039" height="581" alt="image" src="https://github.com/user-attachments/assets/10bd1de6-1ff2-41c2-a29a-fbfc2e690ac3" />
ST中文社区网：<https://www/stmcu.rog.cn>

### STM32选型
由高到低，由大到小

## 学会查看数据手册
<img width="1474" height="651" alt="image" src="https://github.com/user-attachments/assets/f79ceb1b-ecc0-481b-b97a-a54943d35a69" />
芯片的基本参数（STM32F103ZET6为例）：

- 主频/FLASH/SRAM：72MHz/512KB/64KB
- 工作电压/最大电流：2.0~3.6V/150mA
- IO引脚接入电压范围：COMS端口 （-0.3V-3.6V）；兼容5V端口 （-0.3V-5.5V）

STM引脚类型：电源引脚、晶振引脚、复位引脚、下载引脚、BOOT引脚、GPIO引脚（以B开头）
<img width="1260" height="314" alt="image" src="https://github.com/user-attachments/assets/52f6f007-8314-44cc-80d8-6fc2c7ac116d" />
***

## 最小系统和IO分配
### 最小系统
保证MCU正常工作的最小电路组成单元，包括有：STM32F、电源电路、晶振电路、复位电路、BOOT启动电路、下载调试电路、其他电路

1.复位电路：STM32复位引脚NRST保持低电平状态时间1~4.5ms即可复位

2.<img width="1070" height="684" alt="image" src="https://github.com/user-attachments/assets/26979446-e146-4ea1-8b43-e620d62a060c" />
<img width="539" height="609" alt="image" src="https://github.com/user-attachments/assets/4106c5b3-abb3-4e7a-8ca6-523ac66f1c28" />

3.晶振电路：
<img width="1763" height="918" alt="image" src="https://github.com/user-attachments/assets/bf2ce650-1cc8-4189-aa71-71ae899021c5" />

4.下载调试电路：
<img width="1722" height="731" alt="image" src="https://github.com/user-attachments/assets/c394c39d-062c-4ab6-9605-60e479085396" />
### IO分配
优先分配特定外设IO，然后分配通用IO，最后微调

微调主要包括两部分：
- 当IO不够用时，通用GPIO和特定外设可能要共用IO口
- 为了方便布线，可能要调整某些IO口的位置

这两点要做到：尽可能多的可以同时使用所有功能，尽可能方便布线
## STM32系统框架
### Cortex M内核&芯片
<img width="844" height="552" alt="image" src="https://github.com/user-attachments/assets/b298ab8f-39b1-4eed-b5e7-961a9906f21f" />


### F1系统架构
4个主动单元+4个被动单元

|主动单元|被动单元|
|---|---|
|Cortex M3内核DCode总线（D-Bus）|内部FLASH|
|Cortex M3内核 系统总线（S-Bus）|内部SRAM|
|通用DMA1|FSMC|
|通用DMA2|AHB到APB的桥，它连接所有的APB外设|

AHB:高级高性能总线；APB:高级外围总线

ICode总线直接连接到Flash接口，不需要经过总线；

总线时钟频率：
- AHB:72MHz(Max)
- APB1:36MHz(Max)
- APB2:72MHz(Max)

### F4系统架构
8个主控总线+7个被控总线
<img width="1114" height="429" alt="image" src="https://github.com/user-attachments/assets/8c4858b6-1fe8-43d1-91d2-02e63e198f55" />

CCM ARM：只能存数据，优点访问速度快，缺点不支持DMA

总线时钟频率：
- AHB1/2:168/180MHz(Max)
- AHB1:42/45MHz(Max)
- AHB2:84/90MHz(Max)
### F7系统架构
主系统架构：
- 1个AXI转AHB总线桥
  - 1个连接到内嵌FLASH的AXI转64位AHB总线桥
  - 3个连接到AHB总线矩阵的AXI转32位AHB总线桥
- 1个AHB总线矩阵
  - 12个总线主控器
  - 8个总线从控制器

DTCM RAM:即可存放数据，也可存放命令；ITCM RAM：支持CPU时钟速度访问，0个等待周期

总线时钟频率：
- AHB1/2：216MHz(Max)
- APB1:54MHz(Max)
- APB2:108MHz(Max)
### H7系统架构
主系统架构：
- 1个AXI总线矩阵
- 两个AHB总线矩阵
  - D2域的AHB总线矩阵
  - D3域的AHB总线矩阵
- 总线桥
- 域间总线
DTCM：存放数据；ITCM:存放程序

总线时钟频率：
- AHB1/2/3/4:240MHz(Max)
- APB1/2/3/4:120MHz(Max)

## 存储器映射
### SIM32的寻址范围
1.32位的单片机可以有32根地址线（每根地址线有两种状态：导通或不导通）

2.单片机内存地址访问的存储单元是按字节编址的（不是bit）

3.STM32寻址大小：2^32=4G（字节）；STM32寻址范围：0x0000 0000 ~ 0xFFFF FFFF
### 存储器映射
存储器指可以存储数据的设备，本身没有地址信息，对存储器分配地址的过程称为存储器映射。

存储器功能划分（F1为例）
 <img width="1046" height="499" alt="image" src="https://github.com/user-attachments/assets/10522210-b037-4dbc-98f6-fd8ba383aced" />
<img width="1125" height="450" alt="image" src="https://github.com/user-attachments/assets/a55ec18a-ec51-40cd-abb6-4275d8e7b0fd" />
<img width="974" height="225" alt="image" src="https://github.com/user-attachments/assets/bb3831b2-084a-42fa-a50e-69ee0cd9a9a5" />
<img width="971" height="410" alt="image" src="https://github.com/user-attachments/assets/65e94bdf-181a-4431-bf53-bef08387c99f" />

### 寄存器映射
寄存器是单片机内部一种特殊的内存，可以实现对单片机各个功能的控制，即寄存器就是单片机内部的控制机构

STM32寄存器分类
- 内核寄存器
  - 内核相关寄存器
  - 中断控制寄存器
  - 内存保护寄存器
  - 调试系统寄存器
- 外设寄存器：包含GPIO、UART、IIC、SPI、TIM、DMA、ADC、DAC、RTC等各种

寄存器是特殊的存储器，给寄存器地址命名的过程，就叫寄存器映射

寄存器地址计算：将寄存器地址分为三个部分，其地址等于三者相加
- 总线基地址（BUS_BASE_ADDR）
- 外设基于总线基地址的偏移量（PERIPH_OFFSET)【APB1总线的基地址，也叫外设基地址】
- 寄存器相对外设基地址的偏移量（REG_OFFSET)
 寄存器映射方法：

1.单个映射

  <img width="627" height="274" alt="image" src="https://github.com/user-attachments/assets/ee414687-d10a-46b0-9b7e-3c3530e43066" />

2.使用结构体，可以一次映射多个

  <img width="433" height="424" alt="image" src="https://github.com/user-attachments/assets/c6e0ad76-2a8a-44f4-8096-d9a1c48a07f4" />
  
<img width="717" height="383" alt="image" src="https://github.com/user-attachments/assets/f5fbdb9b-bde3-4e7d-846c-d538b81f6135" />

### SYSTEM文件夹介绍
- sys文件夹
<img width="1190" height="460" alt="image" src="https://github.com/user-attachments/assets/5c774dc5-9f86-4fc1-ade7-3f87292fd11a" />

- deley文件夹介绍
  <img width="956" height="222" alt="image" src="https://github.com/user-attachments/assets/a8c6b3da-2d90-42c5-bbf9-7c4e9ce62920" />

  - SysTick工作原理：SysTick，即系统滴答定时器，包含在M3/4/7内核里面，核心是一个24位的递减计数器
  <img width="1112" height="402" alt="image" src="https://github.com/user-attachments/assets/594c12e8-84d9-41dc-ba4b-25c70e34e3ec" />
  
  - SysTick内存器介绍
  <img width="1093" height="376" alt="image" src="https://github.com/user-attachments/assets/b14cf770-66cd-4d6c-95bb-4cacf8acdfb2" />

### STM32启动过程
1.启动模式（F1/F4/F7/H7）也称自举模式：

M3/M4/M7等内核复位后，做的第一件事：从地址0x0000 0000处去除***堆栈指针MSP***的初始值，该值就是栈顶地址；从地址0x0000 0004处取出***程序计数器指针PC***的初始值，该值是复位向量。

F1:在系统复位后，SYSCLK的第四个上升沿，BOOT引脚的值将被封锁

F7：在系统复位后，SYSCLK的第四个上升沿，BOOT引脚的值将被封锁

2.STM32启动过程（内部FLASH启动为例）

***启动流程的五个关键步骤***：
- 上电复位--->硬件初始化
- 执行启动文件（startup_xxx.s）
- SystemInit()---系统时钟初始化
- C/C++运行时初始化
- 进入main函数---用户程序开始
  
即：Reset--->获取MSP值0x0800 0000，获取PC值0x0800 0004--->Reset_Handler--->启动文件startup_stm32xxx.s--->main函数

启动文件介绍：
- 初始化MSP
- 初始化PC
- 设置堆栈大小
- 初始化中断向量表
- 调用初始化函数
- 调用main函数

Reset_Handler函数介绍
<img width="712" height="248" alt="image" src="https://github.com/user-attachments/assets/a1e5056b-e0cc-449c-9ff5-c22defc69bd6" />

|内存|作用|
|---|---|
|栈（Stack)|编译器自动分配和释放，存放函数参数、局部变量等|
|堆（Heap)|程序员分配和释放，如malloc\calloc\realloc等|

函数局部变量较多，嵌套关系复杂时，需加大栈大小（Stack_Size）

***中断向量表***：是一个存储在固定地址的数组，每个元素是一个函数指针（即“向量”），指向对应中断/异常的处理函数

## STM32时钟系统
时钟是具有周期性的脉冲信号，最常用的是占比50%的方波。MCU的所有操作都依赖时钟脉冲驱动。

### 时钟树（F1）
时钟树就是从一个或几个时钟源，通过分频/倍频，生成不同频率的时钟信号，供给各个模块使用

|时钟源名称|频率|材料|用途|
|---|---|---|---|
|高速外部振荡器（HSE）|4~16MHz|晶体/陶瓷|SYSCLK/RTC|
|低速外部振荡器（LSE)|32.768KHz|晶体/陶瓷|RTC|
|高速内部振荡器（HSI）|8MHz|RC|SYSCLK|
|低速内部振荡器（LSI）|40KHz|RC|RTC/IWDG|
|PLL|可变|锁相环倍频输出|系统主时钟SYSCLK|

|符号|作用|
|---|---|
|时钟安全系统（CSS）|如果HSE启动失败，切换到HSI。可进NMI中断|
|自由运行时钟（FCLK）|用于采样中断和调试模块计时，休眠仍有效|

<img width="1294" height="413" alt="image" src="https://github.com/user-attachments/assets/ca9598a8-d9c1-4425-b73b-4297f1b1e33e" />

时钟源、锁相环：HAL_RCC_OscConfig()

系统时钟、总线：HAL_RCC_ClockConfig()

使能外设时钟：__HAL_RCC_PPP_CLK_ENABLE()【必须先使能外设时钟，才能访问外设寄存器】

扩展外设时钟(RTC/ADC/USB):HAL_RCCEx_PeriphCLKConfig

PLL输出频率公式：
- f(VCO) = f(PLL输入) × (PLLN / PLLM)
- f(PLLCLK) = f(VCO) / PLLP
- f(USB OTG FS) = f(VCO) / PLLQ

## GPIO外设原理
GPIO是MCU与外部世界交互的最基本接口，每个GPIO引脚都可以独立配置为***输入或输出***模式，并通过寄存器控制其电平状态。STM32F4的GPIO分为多组（GPIOA~GPIOK），每组最多16个引脚（PA0~PA15PB0~PB15),挂载在AHB1总线上，基地址为0X4002_0000

| 模式       | 方向 | 特点             | 典型应用                |
| -------- | -- | -------------- | ------------------- |
| **浮空输入** | 输入 | 无上拉/下拉，电平由外部决定 | 按键检测（需外部上拉）         |
| **上拉输入** | 输入 | 内部上拉电阻（约40kΩ）  | 按键检测（按键接GND）        |
| **下拉输入** | 输入 | 内部下拉电阻（约40kΩ）  | 按键检测（按键接VCC）        |
| **模拟输入** | 输入 | 直接连到ADC，数字功能关闭 | ADC采样               |
| **推挽输出** | 输出 | 可输出高/低电平，驱动能力强 | LED控制、驱动数字器件        |
| **开漏输出** | 输出 | 只能拉低或释放，需外部上拉  | I2C总线、电平转换          |
| **复用推挽** | 输出 | 由片上外设控制，推挽方式   | USART\_TX、SPI\_MOSI |
| **复用开漏** | 输出 | 由片上外设控制，开漏方式   | I2C\_SDA、SMBus      |

### 核心寄存器详解
#### GPIOx_MODER(模式寄存器)
- 地址偏移：0x00
- 作用：配置每个引脚的工作模式（输入/输出/复用/模拟）
- 位宽：每两位控制一个引脚，共32位

  #### GPIOx_OTYPER(输出类型寄存器)
  - 地址偏移：0x04
  - 作用：配置输出类型（推挽/开漏）
  - 位宽：每1位控制1个引脚
 
  #### GPIOx_OSPEEDR(输出速度寄存器)
  - 地址偏移：0x08
  - 作用：配置输出翻转速度（影响功耗和EMI）
  - 位宽：每2位控制1个引脚

  #### GPIOx_PUPDR(上拉/下拉寄存器)
  - 地址偏移：0x0C
  - 作用：配置内部上拉或下拉电阻
  - 位宽：每2位控制1个引脚

  GPIOx_IDR:0x10,读取引脚当前电平状态（只读），低16位有效，对应16个引脚

  GPIOx_ODR：0x14，直接设置引脚输出电平，低16位有效

  GPIOx_BSRR：0x18,原子操作设置或清除引脚，高16位用于清除，低16位用于设置

  GPIOx_AFRL/AFRH：0x20/0x24，选择引脚的复用功能编号，每4位控制1个引脚





  
