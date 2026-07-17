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














  
