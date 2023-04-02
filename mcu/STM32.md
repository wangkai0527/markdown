[TOC]

# STM32


## STM32CubeIDE
在2018年3月份，当时ST公司已经收购了Atollic公司，TrueSTUDIO 9.0变成一个免费的STM32开发IDE工具。使用STM32CubeMX+TrueSTUDIO 9.0，发现这套工具非常好用，特别是TrueSTUDIO 是基于Eclipse的，具有强大的编辑功能。

2019年4月，ST公司正式发布了STM32CubeIDE 1.0，并且将TrueSTUDIO 设置为 NRND（Not recommended，不推荐使用）。
STM32CubeIDE 就是在TrueSTUDIO基础上推出的一个整合的版本，STM32CubeIDE内部就集成了STM32CubeMX的插件，所以在STM32CubeIDE 里可以使用STM32CubeMX。

[STM32CubeIDE官网下载地址](https://www.st.com/zh/development-tools/stm32cubeide.html)

### 基本介绍
STM32CubeIDE 是一款多功能的多操作系统开发工具，是 STM32Cube 软件生态系统的一部分。
STM32CubeIDE 是一个高级 C / C++ 开发平台，具有基于 STM32 微控制器和微处理器的外设配置、代码生成、代码编译和代码调试等多种功能。它基于 Eclipse / CDT 框架和 GCC 工具链进行开发，基于 GDB 调试器进行调试。它允许集成数百个包含 Eclipse IDE 功能的现有插件。
STM32CubeIDE 集成了 STM32CubeMX 的 STM32 配置和项目创建功能，提供了一体式工具的体验，并节省了安装和开发时间。在选择一个空的 STM32 MCU 或 MPU 之后，或者从预装配的微控制器或微处理器中选择一个单板或一个实例，创建项目并生成初始化代码。在开发过程中，用户可以随时返回到外围设备或中间件的初始化和配置，重新生成初始化代码，而不会对用户代码产生影响。
STM32CubeIDE 包括构建和堆栈分析器，可为用户提供有关项目状态和内存要求的有用信息。
STM32CubeIDE 还包括标准和高级调试功能，包括 CPU 内核寄存器，存储器和外设寄存器的查看窗口，以及实时变量监视，串行线查看器接口或故障分析器。

### 主要特点
集成 STM32CubeMX 的服务：
a. STM32微控制器，微处理器，开发平台和示例项目选择
b. 引脚分配，时钟，外设和中间软件配置
c. 项目创建和初始化代码的生成
d. 软件和中间件由增强的STM32Cube扩展包完成
基于 Eclipse / CDT，支持 Eclipse 的插件，GNU C / C ++中 Arm 工具链和 GDB 调试器
其他高级调试功能：
a. CPU 内核，外设寄存器和内存视图
b. 变量实时查看视图
c. 系统分析和实时跟踪（SWV）
d. CPU 故障分析工具
支持 ST-LINK（STMicroelectronics）和 J-Link（SEGGER）调试探针
可从 Atollic TrueSTUDIO 和 AC6 系统工作台导入STM32（SW4STM32）工程或项目
支持多种操作系统：Windows，Linux，MacOS。（仅支持 64 位版本）


### 创建工程
1.准备单片机、ST-LINK、连接线、电源线，连接好，打开电源
2.打开STM32CubeIDE创建工程
3.选择单片机型号，这里我的开发板是STM32F103ZETx
4.打开.ioc文件，配置STLink（不配置不能在线调试，还会导致FLASH锁死）
Pinout & Configuration -> System Core -> SYS -> Debug --> Serial Wire --> Generate Code
5.Debug -> STLINKUpgrade

开发板stm32f103c8
RCC设置，选择HSE(外部高速时钟)为Crystal/Ceramic Resonator(晶振/陶瓷谐振器)
时钟配置采用图形配置，直观简单。各个外设时钟一目了然。STM32最高时钟为72M，此处只有在HCLK处输入72，软件即可自动配置。（RCC选择外部高速时钟）。
PLL Source Mux选择HSE
*PLLMul选择X9
System Clock Mux选择PLLCLK
HCLK(MHz)设置72
APB1 Prescaler选择/2



***注意：一定要在USER CODE BEGIN和USER CODE END之间写代码，否则重新生成工程代码会覆盖！！！***

|   文件        |   对应工程    |
|   :---:   |   :---:    |
|.cproject   |STM32CubeIDE工程|
|.project    |TrueSTUDIO工程|
|.ioc        |STM32CubeMX工程|


### Debug工程
Debug Configurations -> STM32 C/C++ Application -> 调试器 -> 调试探头选择ST-LINK (ST-LINK GDB server)



#### 实现Led灯的交替闪烁
如果没有可控制的Led，可以找两个gpio脚实现高低电平交替

1.查看开发板开发手册，找到对应的GPIO脚，例如:
Led0 -> PB5
Led1 -> PE5
2.找到并点击Pinout view上的PB5，选择GPIO_Output




task1：点亮gc07s1
task2：usb虚拟串口实现单片机与pc机通信

spi点亮gc07S1下一步任务
1.对读gc07S1读写功能进行封装（不用dma） sensor_read_reg（uint16_t *addr,uint8_t *val）; sensor_write_reg（uint16_t *addr,uint8_t val）; 
2.调通spi dma功能并封装
3.实现多次写入寄存器功能（load setting）

usb虚拟串口下一阶段的任务，定制一个私有协议
1.ready mode
  上位机向单片机发送3个 byte的数据
  byte[0] = 0x01（0x01表示ready状态）,byte[1] = size >> 8,byte[2] = size (size 为下一次通信写入数据的长度)
  单片机向上位机回复一个byte = 0x32 表示通信成功
  单片机状态标识为切换成data mode
2.data mode
  上位机发送byte分析
  Byte[0] 为data mode 标识符
  如果第一次通信成功（单片机状态标识为切换成data mode）且上位机第二次发送的byte[0] = 0x02,则开始解析byte[1]
  Byte[1]为通信类型 例如：0x00 亮红灯 0x01亮绿灯 0x02 SPI 操作



