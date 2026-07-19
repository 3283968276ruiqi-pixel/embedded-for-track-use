# MCU内核架构（Day1）
<https://www.bilibili.com/video/BV1Lq4y1E7Yt/> 第五讲、第六讲

b站搜索：《Cortex-M3权威指南 ARM Cortex-M3 架构》，前三章，补充Cortex-M3架构

ARM是设计公司，不生产芯片，只设计处理器架构和核心蓝图；Cortex是ARM公司对其处理器核心的品牌命名，
Cortex-A是高性能应用处理器，Coetex-R是实时处理器，Coetex-M是微控制器级别；
MCU是一类芯片的统称，STM32103是一款基于Cortex-M3的MCU芯片。

## Cortex内核
Cortex-M3芯片结构（所有的都由芯片制造商设计开发）:
- Cortex-M3内核、调试系统（由ARM设计）
- 外设、存储器、时钟和复位、I/O

STM32命名规则：（以STM32F103C8T6为例）
- STM32=基于ARM的32位微控制器
- F=通用类型
- 101=基本型；102=USB基本型，USB 2.0全速设备；103=增强型；105或107=互联型
- T=36脚（引脚）；C=48,R=64;V=100;Z=144
- 4=16K字节的内存存储器；6=32K字节的内存存储器；8=64K字节的内存存储器；B=128K；C=256K;D=384K;E=512K
- H=BGA;T=LQFP;U=VFQFPN（封装）
- 6=工业级温度范围，-40°C~85℃
## STM32芯片解读
STM32最小系统：供电、复位、时钟：外部晶振（2个）、Boot启动模式选择、下载电路（串口/JTAG/SWD）、后备电池
