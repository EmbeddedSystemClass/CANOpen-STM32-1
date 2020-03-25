# Development Environment Setting
## Install Extension
1. C/C++ IntelliSense, debugging, and code browsing.
2. Cortex-Debug
3. gitignore
4. Mardown Preview
5. Prettier Code formatter using prettier
6. vscode-icons
7. Python
## Extension Config
1. Coretex-Debug
Create launch.json file, with this config.
``` json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceRoot}",
            "executable": "./build/CANOpen-STM32.elf",
            "request": "launch",
            "type": "cortex-debug",
            "servertype": "openocd",
            "device": "STM32F446RE",
            "configFiles": [
                "./board/stm32f4x.cfg"
            ]
        }
    ]
}
```
meanwhile, copy /usr/share/openocd/* files, including stm32f4x.cfg, swj-dp.tcl mem_helper.tcl to local directory.

Fix the directory path problem
```
#
# stm32 devices support both JTAG and SWD transports.
#
source ./board/swj-dp.tcl
source ./board/mem_helper.tcl
```
And put the next code at the beginning of stm32f4x.cfg, for config the st-link config
```
# script for stm32f4x family
interface hla
hla_layout stlink
# lsusb, then can known the st-link usb-id and desc
hla_device_desc "ST-LINK/V2.1"
hla_vid_pid 0x0483 0x374b
adapter_khz 1800
transport select hla_swd

```
2. C/C++ Auto-completion
VSCode will automatically promote add c_cpp_properties.json to config project. Add include path to the json files.
``` json
{
    "configurations": [
        {
            "name": "STM32CubeMX",
            "includePath": [
                "${workspaceFolder}/Inc",
                "${workspaceFolder}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2",
                "${workspaceFolder}/Middlewares/Third_Party/FreeRTOS/Source/include",
                "${workspaceFolder}/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F",
                "${workspaceFolder}/Drivers/STM32F4xx_HAL_Driver/Inc",
                "${workspaceFolder}/Drivers/CMSIS/Device/ST/STM32F4xx/Include"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/clang",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64",
            "compileCommands": "${workspaceFolder}/compile_commands.json"
        }
    ],
    "version": 4
}
```
3. compile_commands.json generation

For linux system, it's quite simple, use bear.

For windows/macOS, compiledb from python is another choice.
``` shell
$ bear make
or
$ compiledb make -j8

By default, compiledb make generates the compilation database and runs the actual build command requested (acting as a make wrapper), the build step can be skipped using the -n or --no-build options.

compiledb forwards all the options/arguments passed after make subcommand to GNU Make, so one can, for example, generate compile_commands.json using core/main.mk as main makefile (-f flag), starting the build from build directory (-C flag):

```
---
# STM32F446 Nucleo Perpherial Config
## UART
1. Basic method, poll status
2. Interrupt handler
3. DMA Transfer
## DMA
For the uart, there are two interrupts happened, half transmit and transmit completed.
https://stackoverflow.com/questions/43298708/stm32-implementing-uart-in-dma-mode
From up link, has a clear explaination about how it works.

## Timer
The Basic Timers (BT) TIM6, TIM7, TIM14, etc (1°) are the most simple timers available in the STM32 portfolio.

The BT are 16 bit timer.

The BT are UP timer only.

The BT my be used in DMA and/or under Interrupt.

The BT has the capabilities show below.

Plese refer to the AN4013 for more info on the STM32 Timers.
![20200324140124.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324140124.png)
![20200324140209.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324140209.png)

For this application, use basic timer modes.

Hardware timers keep counting up or down depending on the module until the timer period is reached. Then the timer is reset:
![update events.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324134420.png)

We will use the timer to keep our LED on 80% of the time by setting a period of 500, turning it on when the counter reaches 400 and turning it off when it reaches 500:
![20200324134510.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324134510.png)

http://www.emcu.eu/stm32-basic-timer/ make a good explanation about the timer.

## CAN
Basic knowledge at here
https://www.notion.so/c2real/CAN-Basic-Knowledge-22b2ce481c124ec38f41f09ae16f0a74

ISO11898标准，该标准的物理层特征如下图所示：
![20200325100050.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325100050.png)
从该特性可以看出，显性电平对应逻辑0，CAN_H和CAN_L之差为2.5V左右。而隐性电平对应逻辑1，CAN_H和CAN_L之差为0V。在总线上显性电平具有优先权，只要有一个单元输出显性电平，总线上即为显性电平。而隐形电平则具有包容的意味，只有所有的单元都输出隐性电平，总线上才为隐性电平（显性电平比隐性电平更强）。另外，在CAN总线的起止端都有一个120Ω的终端电阻，来做阻抗匹配，以减少回波反射。

CAN协议是通过以下5种类型的帧进行的：
* 数据帧
* 摇控帧
* 错误帧 
* 过载帧
* 帧间隔
 
数据帧和遥控帧有标准格式和扩展格式两种格式。标准格式有11 个位的标识符（ID），扩展格式有29 个位的ID。

数据帧一般由7个段构成，即：
（1）       帧起始。表示数据帧开始的段。
（2）       仲裁段。表示该帧优先级的段。
（3）       控制段。表示数据的字节数及保留位的段。
（4）       数据段。数据的内容，一帧可发送0~8个字节的数据。
（5）       CRC段。检查帧的传输错误的段。
（6）       ACK段。表示确认正常接收的段。
（7）       帧结束。表示数据帧结束的段

**Standard frame**
![20200325142124.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325142124.png)
SOF – Start of Frame bit. It indicates  start of message and used to synchronize the nodes on a bus. A dominant bit in the field marks the start of frame.

IDENTIFIER – It serves dual purpose one, to determine which node has access to the bus and second to identify the type of message.

RTR – Remote Transmission Request. It identifies whether it’s a data frame or a remote frame .RTR is **dominant** when it is a data frame and **recessive** when it is a remote frame.

IDE – Identifier Extension. It is used to specify the frame format. Dominant bit is for standard frame and recessive for extended frame.

R0 – Reversed bit.  Not used currently and kept for future use.

EOF– End of Frame (EOF). The  7-bit field marks the end of a CAN frame (message) and disables Bit – stuffing, indicating a stuffing error when dominant.

IFS – Inter Frame Space that specifies minimum number of bits separating consecutive messages. It provides the intermission between two frames and consists of three recessive bits known as intermission bits. This time allows nodes for internal processing before the start of next frame.
![20200325144006.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325144006.png)


**EXTENDED CAN**
![20200325142508.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325142508.png)

SRR- Substitute Reverse Request. The SRR bit is always transmitted as a **recessive** bit to ensure that, in the case of arbitration between a Standard Data Frame and an Extended Data Frame, the Standard Data Frame will always have priority if both messages have the same base (11 bit) identifier. 

![20200325143945.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325143945.png)

CAN2.0A only has standard frame.
Both bits, r1 and r0, were reserved for future use and were kept at a low (dominant) level.
![20200325145349.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325145349.png)

CAN2.0B
The Control Field was “re-designed” (CAN 2.0B) as shown in picture 4.6.2 in order to support co-existence of 11 and 29 bit identifiers on the same CAN bus.
![20200325145430.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325145430.png)
A low (dominant) IDE bit indicates an 11 bit message identifier, a high (recessive) IDE bit indicates a 29 bit identifier.

A **low (dominant) IDE** bit indicates an 11 bit message identifier.
![20200325145633.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325145633.png)

A **high (recessive) IDE** bit indicates a 29 bit identifier.
![20200325145701.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325145701.png)

The 29 bit message identifier consists of the regular 11 bit base identifier and an 18 bit identifier extension. Between the SOF (Start of Frame) bit and the end of the 11 bit (base) message identifier, both frame formats, Standard and Extended, are identical.

Following the 11 bit base identifier, the Extended Format uses an (always recessive) SRR (Substitute Remote Request) bit, which, as its name implies, replaces the regular RTR (Remote Transmission Request). The following IDE (Identifier Extension) bit is also kept at **a recessive level**.

With the use of a recessive SRR plus a recessive IDE bit it is guaranteed that standard message frames (11 bit identifier) will always have higher priority than extended message frames (29 bit identifier) with identical 11 bit base identifier (see also Chapter 6 - Bus Arbitration).

The Control Field of an extended message frame, following the 18 bit extended identifier plus RTR bit, has the format of the original CAN 2.0A standard.
The CAN2.0B frame after arbitration field is still the same as CAN2.0A frame.
Bits r1 and r0 are reserved for future use and are always kept at a dominant level.

![20200325145910.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200325145910.png)



## STM32中的CAN接口
STM32的芯片中具有bxCAN控制器 (Basic Extended CAN)，它支持CAN协议2.0A和2.0B标准。该CAN控制器支持最高的通讯速率为1Mb/s；可以自动地接收和发送CAN报文，支持使用标准ID和扩展ID的报文；外设中具有3个发送邮箱，发送报文的优先级可以使用软件控制，还可以记录发送的时间；具有2个3级深度的接收FIFO，可使用过滤功能只接收或不接收某些ID号的报文；可
配置成自动重发；不支持使用DMA进行数据收发。
![20180804163024150.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20180804163024150.png)
STM32的有两组CAN控制器，其中CAN1是主设备，框图中的“存储访问控制器”是由CAN1控制的，CAN2无法直接访问存储区域，所以使用CAN2的时候必须使能CAN1外设的时钟。

STM32至少配备一个bxCAN(basic extend can )控制器,支持2.0A和2.0B协议,最高数据传输速率可达1M bps,支持11位标准帧格式和29位扩展帧格式的接收和发送,具备三个发送邮箱和两个接收FIFO,此wa此外还有三级可编程滤波器,STM32的bxCAN非常适应CAN总线网络y网络应用发展需求,其主要主要特征如下 :
1. 支持CAN协议2.0A和2.0B主动模式
2. 波特率最高可达1Mbps
3. 支持时间触发通讯功能
4. 数据发送特性:具备三个发送邮箱;发送报文的优先级可以通过软件配置,可记录发送时间的时间戳
5. 数据接收特性:具备三级深度和两个接收FIFO;具备可变的过滤器组,具备可编程标识符列表,可配置FIFO溢出处理方式,记录接收时间的时间戳
6. 报文管理:中断可屏蔽;邮箱单独占有一块地址空间,便于提高软件效率.

STM32 APB1 provide clock to CAN peripheral, 45MHz.
So define 
* prescaler(for quantum) 9
* ts1 = 5
* ts2 = 5
* sw = 4
we can get 500Kbps can bus baudrates.
![2020-03-24-232818_604x431_scrot.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/2020-03-24-232818_604x431_scrot.png)

CAN总线屏蔽滤波
STM32的标识符屏蔽滤波目的是减少了CPU处理CAN通信的开销。STM32的过滤器组最多有28个（互联型），每个滤波器组x由2个32为寄存器，CAN_FxR1和CAN_FxR2组成。
STM32每个过滤器组的位宽都可以独立配置，以满足应用程序的不同需求。根据位宽的不同，每个过滤器组可提供：
* 1个32位过滤器，包括：STDID[10:0]、EXTID[17:0]、IDE和RTR位
* 个16位过滤器，包括：STDID[10:0]、IDE、RTR和EXTID[17:15]位
此外过滤器可配置为，屏蔽位模式和标识符列表模式。
在屏蔽位模式下，标识符寄存器和屏蔽寄存器一起，指定报文标识符的任何一位，应该按照“必须匹配”或“不用关心”处理。

而在标识符列表模式下，屏蔽寄存器也被当作标识符寄存器用。因此，不是采用一个标识符加一个屏蔽位的方式，而是使用2个标识符寄存器。接收报文标识符的每一位都必须跟过滤器标识符相同。

屏蔽滤波模式：
标识符寄存器：0 0 1
屏蔽寄存器： 1 0 1
报文ID号： 0 0/1 1
如果设置标识符寄存器和屏蔽寄存器为001和101；屏蔽滤波模式的作用是如果屏蔽寄存器某位上出现了1，则报文ID号对应的那位要与标识符寄存器那位一致，即“必须匹配”原则，所以标识符寄存器第一位0，报文ID号第一位也必须为0，因为屏蔽寄存器第一位为1，类似的第三位也是这样。如果屏蔽寄存器某位上出现了0，则报文ID号对应的那位可与标识符寄存器那位不一致也可以一致，即“不用关心”原则，第二位由于屏蔽寄存器上为0，所以报文ID号可以与标识符寄存器上的0一致也可以不一致，故报文ID号第二位为0/1。所以002号（010）可以接受来自001号（001）和003号（011）的报文。

标识符列表模式：将设置的屏蔽寄存器改为标识符寄存器
标识符寄存器：0 0 1
标识符寄存器： 0 0 1
报文ID号： 0 0 1
如果设置2个标识符寄存器为001和001；报文ID号必须与这两个标识符寄存器所对应的位相等。所以002号CAN只能接受001号的报文

The bxCAN provides up to 28 scalable/configurable identifier filter banks in dual CAN
configuration, for selecting the incoming messages, that the software needs and discarding
the others.

Two receive FIFOs are used by hardware to store the incoming messages. Three complete
messages can be stored in each FIFO. The FIFOs are managed completely by hardware.

https://blog.csdn.net/qq_29413829/article/details/53230716
https://blog.csdn.net/qq_36355662/article/details/80607453?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task
http://www.copperhilltechnologies.com/can-bus-guide-higher-layer-protocols/

    
# CANOpen Protocol
https://wenku.baidu.com/view/1bd617d081eb6294dd88d0d233d4b14e85243e96.html
