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
ST中文社区网:<https://www/stmcu.rog.cn>

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

## 最小系统和IO分配



  
